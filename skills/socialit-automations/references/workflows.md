# Workflow reference

`list-workflow-node-types` returns the live catalog: each type with its group, description, output `handles`, `config_schema`, and the networks it supports. Use it when this page and the server disagree.

## Graph shape

- **Node**: `{ "id", "type", "label", "position": { "x", "y" }, "config" }`. `id` is 1 to 64 letters, digits, `_`, or `-`. Every string in `config` can use `{{variables}}`.
- **Edge**: `{ "id", "source", "target", "source_handle" }`. `source_handle` is `null` for a node with one output, or the name of the branch (see the handles below).
- `entry_node_id` is the trigger node. Up to 200 nodes and 400 edges.
- `account_id` is required when the trigger reacts to messages or comments, or when the graph sends messages.
- `nodes` and `edges` are replaced as a whole on `update-workflow`, and only while `draft` or `paused`.

`create-workflow` checks the structure (unique IDs, known types, edges between real nodes, valid handles) and answers 422 with errors keyed like `nodes[3].config.url`. `activate-workflow` also needs exactly one trigger, every node reachable and valid, and an account when the graph messages (422 `workflow_invalid` with `errors`).

## Triggers

Set on the `trigger` node as `config.trigger_type`.

| `trigger_type` | Starts a run when | Config |
| --- | --- | --- |
| `inbound_message` (default) | A DM arrives on the account | `only_first_message` |
| `keyword` | A DM matches a keyword | `keywords`, `match_type` (`any`, `contains`, `exact`, `regex`) |
| `comment` | Someone comments on the account's posts | `platform_post_id` for one post, `keywords`, `match_type` |
| `story_mention` | The account is mentioned in a story | |
| `referral` | A conversation opens from a referral | `ref`, `ref_match` (`exact`, `prefix`, `any`) |
| `reaction` | Someone reacts to a message | `emojis` |
| `contact_tag_added` | A tag is added to a contact | `tag` (empty means any tag) |
| `contact_field_changed` | A custom field value changes | `field` slug (empty means any field) |
| `schedule` | On a timer | `schedule: { "every_minutes", "cron", "timezone" }` and `audience: { "tags", "is_subscribed" }` |
| `manual` | `start-workflow-execution` is called | |
| `api_call` | `trigger-workflow` is called | |

`cooldown_hours` (1 to 720) on any trigger keeps a workflow from restarting for the same contact within that window.

A `schedule` trigger with an `audience` starts one run per matching contact every time it fires, so it behaves like a recurring broadcast. Count the audience first (`list-contacts` with the same tags) and confirm both the count and the schedule before activating. Without an `audience`, it starts one run with no contact.

## Nodes

| Type | Does | Config | Handles |
| --- | --- | --- | --- |
| `send_message` | DMs the contact on the workflow account | `message_type` (`text`, `media`, `buttons`, `quick_replies`), `text`, `media: { media_type, url, caption }`, `buttons` (1 to 3), `quick_replies` (up to 13) | one |
| `reply_to_comment` | Public reply to the comment that started the run (Instagram, Facebook, Threads, TikTok, YouTube, LinkedIn, Bluesky, Reddit) | `text` | one |
| `private_reply` | One DM to the commenter (Instagram, Facebook; once per comment, within 7 days) | `text`, `buttons` | one |
| `wait_for_reply` | Waits until the contact answers or the timeout | `timeout_minutes` (1 to 43,200), `save_as` | `reply`, `timeout` |
| `delay` (alias `wait`) | Waits a fixed time | `delay_minutes` (1 to 43,200) | one |
| `condition` | Branches on rules; the first match wins | `rules: [{ "id", "variable", "operator", "value" }]` | each rule `id`, and `default` |
| `a_b_split` | Random split | `percentage` sent to `a` (default 50) | `a`, `b` |
| `set_variable` | Sets run variables | `assignments: [{ "name", "value" }]` | one |
| `add_tag`, `remove_tag` | Tags or untags the contact | `tag` | one |
| `set_field` | Sets a custom field | `field` (slug), `value` | one |
| `enroll_sequence` | Enrolls the contact in a sequence | `sequence_id` | `success`, `error` |
| `http_request` (alias `webhook`) | Calls an HTTPS endpoint; the response is saved as `{ status, ok, body }` | `url`, `method`, `headers`, `body_template`, `save_as`, `timeout_ms` (up to 15,000) | `success`, `error` |
| `ai` (alias `ai_reply`) | Generates text with Claude | `system_prompt`, `user_prompt_template` (default `{{last_message}}`), `max_tokens`, `save_as` (default `ai`), `send_reply`, `output_type` (`text`, `json`) | `success`, `error` |
| `assign_conversation` | Assigns the conversation to a teammate | `assignee_id` (`usr_...`), `note` | one |
| `handoff` | Assigns to a person and ends the run as `exited` | `assignee_id`, `note` | none |
| `end` | Ends the run as `completed` | | none |

The first `send_message` of a run started by a comment goes out as the private reply. Message nodes follow the same network rules as broadcasts: on Messenger and Instagram, nothing is sent outside the 24 hour window.

`ai` with `send_reply: true` sends generated text to customers with no human review. Agree on the system prompt with the user, test it on their own contact, and suggest `send_reply: false` (saving the text for a later node or a person) when they are unsure. `http_request` headers are stored in the workflow; use a dedicated, narrowly scoped token for the user's API.

## Branching example

```json
{
  "nodes": [
    { "id": "t1", "type": "trigger", "config": { "trigger_type": "inbound_message", "only_first_message": true } },
    { "id": "c1", "type": "condition", "config": { "rules": [{ "id": "vip", "variable": "contact.tags", "operator": "has_tag", "value": "vip" }] } },
    { "id": "a1", "type": "assign_conversation", "config": { "assignee_id": "usr_...", "note": "VIP wrote in" } },
    { "id": "m1", "type": "send_message", "config": { "message_type": "text", "text": "Thanks {{contact.first_name}}, we will answer shortly." } }
  ],
  "edges": [
    { "id": "e1", "source": "t1", "target": "c1", "source_handle": null },
    { "id": "e2", "source": "c1", "target": "a1", "source_handle": "vip" },
    { "id": "e3", "source": "c1", "target": "m1", "source_handle": "default" }
  ],
  "entry_node_id": "t1"
}
```

## Condition operators

`equals`, `not_equals`, `contains`, `not_contains`, `starts_with`, `ends_with`, `matches` (regular expression), `exists`, `not_exists`, `greater_than`, `less_than`, `greater_or_equal`, `less_or_equal`, `before`, `after` (dates), `has_tag`, `not_has_tag` (on `contact.tags`), `is_true`, `is_false`. Text comparisons ignore case. The same operators work in sequence step conditions.

## Run variables

`last_message`, `trigger_text`, `conversation_id`, `contact.id`, `contact.name`, `contact.first_name`, `contact.email`, `contact.company`, `contact.handle`, `contact.tags`, `contact.fields.<slug>`, `contact.is_subscribed`, `comment.id`, `comment.text`, `comment.platform_post_id`, `referral.ref`, `referral.source`, `postback.payload`, `quick_reply.payload`, `reaction.emoji`, `story.id`, `story.url`, `trigger.type`, plus the `variables` passed to `trigger-workflow` or `start-workflow-execution`, `save_as` results, and `set_variable` names.

## Runs

- Workflow `status`: `draft`, `active`, `paused`. `get-workflow` also returns `version` and `stats` (`total_started`, `total_completed`, `total_exited`, `total_failed`, `running`).
- Run `status`: `running`, `waiting` (see `waiting_for` with `kind`, `node_id`, `until`), `completed`, `exited`, `failed` (`last_error`).
- `list-workflow-executions` filters by `status` and `contact_id`. `list-workflow-execution-events` returns the timeline: each step's `action`, `status`, `node_id`, `node_type`, `source_handle`, `duration_ms`, and `error_message`.
- `cancel-workflow-execution` stops one run (it ends as `exited`); confirm first.

## Rules worth knowing

- A matching comment automation answers a comment before any `comment` workflow.
- Messages sent by workflows, sequences, broadcasts, and comment automations never start workflows.
- A redelivered inbound event never starts the same workflow twice.
- `trigger-workflow` needs an `active` workflow (409 `workflow_not_active`) whose trigger is `api_call` (422 `trigger_mismatch`). Over REST, an `Idempotency-Key` header makes the call safe to retry; reusing the key later returns 409 `duplicate_trigger`.
- Webhook events: `workflow.triggered`, `workflow.completed`, `workflow.failed` (with `error.node_id` and `error.message`).
