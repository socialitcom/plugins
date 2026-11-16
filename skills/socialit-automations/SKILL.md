---
name: socialit-automations
description: This skill should be used when the user asks to "add a contact", "tag these contacts", "import contacts", "create a custom field", "send a broadcast", "message everyone tagged vip", "schedule a broadcast", "build a drip sequence", "enroll contacts in a sequence", "create a workflow", "auto-reply to keyword DMs", or "why did my workflow fail" in Socialit. Covers contacts, channels, tags, and custom fields, broadcasts with an audience preview, sequences, and event-driven workflows with versions and runs, with consent and network messaging rules.
---

# Socialit contacts and automations

Everyone who writes to a connected account becomes a contact. Contacts feed **broadcasts** (one-off bulk DMs), **sequences** (timed drips), and **workflows** (event-driven graphs). Comment-to-DM rules are covered in the `socialit-inbox` skill. For workspaces, permissions, IDs, and errors, follow the `socialit` skill.

| Resource | Toolset | Plan |
| --- | --- | --- |
| Contacts, channels, tags | `contacts` | Every plan |
| Custom fields | `custom_fields` | Every plan |
| Broadcasts | `broadcasts` | Pro and above |
| Sequences, workflows | `sequences`, `workflows` | Pro and above |

On a lower plan the tools return 403 `plan_required`.

## These message real people

Broadcasts, sequences, and active workflows send DMs to many real people at once, under the brand's name, and sent messages cannot be recalled.

- **Nothing sends without a yes.** Before `send-broadcast`, `schedule-broadcast`, `activate-sequence`, `enroll-sequence-contacts`, `activate-workflow`, `start-workflow-execution`, or `trigger-workflow`, show the sending account, the exact message, and the audience size, then wait for an explicit yes.
- **Consent.** Message only people who wrote to the account or opted in. Do not import purchased or scraped lists to message them. Never set `is_subscribed: true` for someone who opted out unless they asked to opt back in.
- **Network rules.** Messenger and Instagram allow messages only within 24 hours of the contact's last message. Socialit skips the others with `outside_messaging_window` instead of sending. `HUMAN_AGENT` stretches this to 7 days, but Meta reserves it for replies by a person on the team, so never use it to push a broadcast or sequence.
- **Tags and fields can start workflows.** `update-contact-tags`, `bulk-tag-contacts`, and field changes fire `contact_tag_added` and `contact_field_changed` triggers. Before bulk tagging, check `list-workflows` for an active workflow on that tag.
- **Destructive**: `bulk-delete-contacts`, `delete-contact`, `delete-custom-field` (removes every stored value), `cancel-broadcast`, `cancel-workflow-execution`, and the other delete tools. Name what will go, with counts, and confirm.

## Contacts

- `list-contacts` filters by `profile_id`, `account_id` (has a channel on that account), `platform`, `search` (name, email, company), `tag`, `tags` (comma separated, any of them), `is_subscribed`, and `is_blocked`. Custom fields go in `filters`, for example `{ "field": "custom_fields.plan", "condition": "exact", "value": "pro" }`.
- `get-contact` returns tags, custom field values, channels, and message counters.
- `create-contact` needs `name`; add `email`, `company`, `notes`, `tags`, `custom_fields`, and optionally a channel (`account_id` plus `platform_identifier`). 409 `contact_channel_exists` means that identity already belongs to a contact.
- `bulk-create-contacts` takes up to 1,000 rows. Duplicates (same channel identity, or same email in the profile) are skipped and their tags merged; row errors come back in `errors`.
- `update-contact` with `tags` **replaces** the whole set. To add or remove a few, use `update-contact-tags` (`add`, `remove`) or `bulk-tag-contacts` (up to 1,000 contacts). `is_subscribed: false` unsubscribes the contact and ends sequences that exit on unsubscribe.
- Channels are the contact's identities per network: `list-contact-channels`, `create-contact-channel`, `update-contact-channel` (`is_subscribed` opts one channel in or out), `delete-contact-channel`. Each shows `last_inbound_at` and `messaging_window_open`.
- `list-contact-tags` lists the tags in use, most used first.

## Custom fields

`create-custom-field` with `name` and `type`: `text`, `number`, `date`, `boolean`, or `select` (with `options`). The `slug` defaults to the name in snake_case, and type and slug cannot change later. Set a value with `set-contact-field-value` (`id`, `slug`, `value`; type checked), clear it with `clear-contact-field-value`, or pass `custom_fields` to `update-contact` (`null` clears).

## Broadcasts

Flow: **draft, preview the audience, confirm, send or schedule, monitor.**

1. `create-broadcast` with the sending `account_id`, a `name`, `message` (`text` up to 4,096 characters, optional `attachments`), and `segment_filters` (`tags`, `tags_match` of `any` or `all`, `exclude_tags`, `custom_fields`). It starts as `draft`.
2. `preview-broadcast-audience` with the same `account_id` and `segment_filters` returns `count` and a `sample`. Always run it. With no filters, the audience is **every subscribed contact** with a channel on that account.
3. Show the user the account, the message, and the count. On Messenger and Instagram, say that contacts outside the 24 hour window are counted but will be skipped.
4. After an explicit yes, `send-broadcast`, or `schedule-broadcast` with a future `scheduled_at` (ISO 8601). Treat scheduling like sending.
5. Monitor with `get-broadcast` (status and counters) and `list-broadcast-recipients` (per recipient `status` and `error_code`).
6. `cancel-broadcast` works while `draft`, `scheduled`, or `sending`: pending recipients become `skipped`, and messages already sent stay sent.

Personalize with `{{first_name}}`, `{{name}}`, `{{username}}`, and `{{fields.<slug>}}`. Missing values render as empty text, so write messages that read well without them. Recipient lists, statuses, and pacing are in `references/broadcasts-and-sequences.md`.

## Sequences

1. `create-sequence` with `account_id`, `name`, and `steps`: `[{ "order": 1, "delay_minutes": 0, "message": { "text": "..." } }]`, up to 50 steps, each with an optional `condition`. The first delay counts from enrollment, each later one from the previous step. Keep `exit_on_reply` and `exit_on_unsubscribe` on (the defaults).
2. Review the steps with the user, then `activate-sequence`. Activating also resumes paused enrollments and sends overdue steps at once.
3. `enroll-sequence-contacts` with `contact_ids` (up to 1,000) starts each contact's clock; contacts already enrolled are skipped. Confirm the list first.
4. `pause-sequence` stops sending. Steps can only change while `draft` or `paused`. `list-sequence-enrollments` shows progress and `exit_reason`; `unenroll-sequence-contact` takes one contact out.

## Workflows

A workflow is one account's graph of `nodes` and `edges` with exactly one `trigger` node, for example a keyword DM, a comment, a new tag, a schedule, or an API call.

1. `list-workflow-node-types` returns every node type with its config schema and output handles.
2. `create-workflow` with `account_id`, `name`, `nodes`, `edges`, and `entry_node_id`. It is created as `draft`, and the structure is validated.
3. `get-workflow`, then explain the flow to the user in plain words.
4. Test with `start-workflow-execution` (works on drafts). It runs the real nodes and messages the real contact, so use the user's own contact (`contact_id`), with `text` to stand in for the incoming message.
5. `activate-workflow` after a yes. From then on it runs on every matching event. 422 `workflow_invalid` lists what to fix.
6. `pause-workflow` stops new runs while running ones finish. The graph can only change while `draft` or `paused`; every change is a version (`list-workflow-versions`, `get-workflow-version`, `restore-workflow-version`). `duplicate-workflow` copies it as a draft.
7. Runs: `list-workflow-executions`, `get-workflow-execution`, `list-workflow-execution-events` (the timeline, with the failing node), and `cancel-workflow-execution`.
8. `trigger-workflow` starts an active workflow whose trigger is `api_call`, with `contact_id` or `conversation_id` and optional `variables`.

```json
{
  "account_id": "sa_...",
  "name": "Price keyword",
  "entry_node_id": "t1",
  "nodes": [
    { "id": "t1", "type": "trigger", "config": { "trigger_type": "keyword", "keywords": ["price"], "match_type": "contains" } },
    { "id": "m1", "type": "send_message", "config": { "message_type": "text", "text": "Hi {{contact.first_name}}, prices start at $25." } }
  ],
  "edges": [{ "id": "e1", "source": "t1", "target": "m1", "source_handle": null }]
}
```

Triggers, nodes, branches, operators, and variables are in `references/workflows.md`. A matching comment automation answers a comment before any `comment` workflow, and messages sent by automations never start workflows.

## Debugging

| Symptom | Check |
| --- | --- |
| Recipients `skipped` | `error_code`: `outside_messaging_window`, `unsubscribed`, `blocked`, `contact_deleted`, or `cancelled` |
| Recipients `failed` with `account_needs_reconnect` | Reconnect the sending account (see `socialit-get-started`) |
| `send-broadcast` returns 422 `no_recipients` | The segment matches nobody; run `preview-broadcast-audience` |
| 409 `broadcast_not_editable` | The broadcast is already sending or finished |
| 409 `sequence_not_active` on enroll | Activate the sequence first |
| 409 `sequence_not_editable` or `workflow_not_editable` | Pause it first |
| A workflow never starts | It is not `active`, the trigger does not match, the trigger `cooldown_hours` applies, or a comment automation answered first |
| A run failed | `list-workflow-execution-events` names the node and the error |
