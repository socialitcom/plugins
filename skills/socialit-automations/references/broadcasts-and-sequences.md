# Broadcasts and sequences reference

Both send DMs from one connected account to contacts that have a subscribed channel on that account. Channels exist on Instagram, Facebook, X, Bluesky, Reddit, Discord, Mastodon, and TikTok.

## Message

```json
{
  "text": "Hi {{first_name}}, our spring menu is live.",
  "attachments": [{ "type": "image", "url": "https://example.com/menu.jpg" }]
}
```

- `text` up to 4,096 characters; up to 10 `attachments` (`type`: `image`, `video`, `audio`, `file`).
- Variables: `{{first_name}}`, `{{name}}`, `{{username}}`, `{{email}}`, `{{company}}`, `{{fields.<slug>}}`, and the same under `contact.` (for example `{{contact.tags}}`). Missing values render as empty text.
- `message_tag` is only for replies by a person on the team (`HUMAN_AGENT`, Messenger and Instagram, up to 7 days after the contact's last message). Leave it out of marketing messages.

## Broadcasts

### Segment filters

| Field | Meaning |
| --- | --- |
| `tags` | Contacts with these tags |
| `tags_match` | `any` (default) or `all` of `tags` |
| `exclude_tags` | Leave out contacts with any of these tags |
| `is_subscribed` | Defaults to subscribed contacts only |
| `custom_fields` | `{ "<slug>": value }`, exact match |

Blocked contacts and channels that opted out are never included. `preview-broadcast-audience` (`account_id`, `segment_filters`) returns `count` and a small `sample` (`contact_id`, `name`, `platform_identifier`).

### Recipients

- With no explicit recipients, `send-broadcast` resolves the segment at send time (422 `no_recipients` when nothing matches).
- `add-broadcast-recipients` with `contact_ids` adds specific contacts, or `use_segment: true` adds the segment now. Contacts without a subscribed channel on the account are skipped. It returns `added`, `skipped`, and `recipient_count`. Only while `draft` or `scheduled`.
- `list-broadcast-recipients` filters by `status` and `contact_id`.

### Statuses

| Broadcast `status` | Meaning |
| --- | --- |
| `draft` | Editable, not sent |
| `scheduled` | Sends at `scheduled_at`; still editable |
| `sending` | Going out in paced batches; can be cancelled |
| `completed` | Finished |
| `failed` | Every recipient failed, nothing matched, or the account was disconnected (`last_error`) |
| `cancelled` | Stopped; pending recipients were skipped |

| Recipient `status` | Meaning |
| --- | --- |
| `pending` | Waiting for its batch |
| `sent`, `delivered`, `read` | Sent, and later confirmed by the network where it reports it |
| `skipped` | Not sent by design: `error_code` such as `outside_messaging_window`, `unsubscribed`, `blocked`, `message_tag_not_allowed`, `contact_deleted`, `cancelled` |
| `failed` | The network refused it or the account needs reconnecting (`error`, `error_code`) |

The broadcast has matching counters: `recipient_count`, `pending_count`, `sent_count`, `delivered_count`, `read_count`, `failed_count`, `skipped_count`.

### Pacing

Messages per minute per sending account: Facebook 300, Instagram 100, other networks 60. `rate_limit_per_minute` (1 to 6,000) overrides it; staying at or below the default protects the account from network rate limits.

### Edits and stops

- `update-broadcast` only while `draft` or `scheduled` (409 `broadcast_not_editable`).
- `delete-broadcast` is refused while `sending` (409 `broadcast_sending`); cancel first.
- `cancel-broadcast` while `draft`, `scheduled`, or `sending` (409 `broadcast_not_cancellable` otherwise).

## Sequences

### Steps

```json
{
  "account_id": "sa_...",
  "name": "Welcome drip",
  "steps": [
    { "order": 1, "delay_minutes": 0, "message": { "text": "Thanks for writing, {{first_name}}!" } },
    { "order": 2, "delay_minutes": 1440, "message": { "text": "Here is our guide." },
      "condition": { "variable": "contact.tags", "operator": "not_has_tag", "value": "customer", "on_false": "exit" } }
  ]
}
```

- `order` must be unique; `delay_minutes` from 0 to 525,600 (one year). The first delay counts from enrollment, each later one from the previous step.
- `condition` (optional) is checked before the step is sent: `variable`, `operator`, `value`, and `on_false`: `skip` (default, go to the next step) or `exit` (end the enrollment, recorded as `exit_reason: "manual"`). Operators are listed in `references/workflows.md`.
- On Messenger and Instagram, a step that comes due more than 24 hours after the contact's last message cannot be sent, and the enrollment ends with `exit_reason: "failed"` (`last_error` says why). Keep drips there within a day of the contact writing.
- Network errors that are temporary (rate limits, outages) are retried later; other send errors end the enrollment.

### Lifecycle

| Sequence `status` | Allowed |
| --- | --- |
| `draft` | Edit steps; activate (needs at least one step). Enrolling is refused with 409 `sequence_not_active` |
| `active` | Enroll; pause. Steps cannot change (409 `sequence_not_editable`) |
| `paused` | Edit steps; enroll; activate again (overdue steps go out at once) |

- `enroll-sequence-contacts`: `contact_ids` (up to 1,000), optional `channel_ids` to pin a channel. Returns `enrolled`, `skipped` (already enrolled), `failed`, and per contact `results` with `error` (for example `contact_not_found`, `contact_in_other_profile`).
- Enrollment `status`: `active`, `paused`, `completed`, `exited`. `exit_reason`: `completed`, `replied`, `manual`, `failed`, `unsubscribed`, `sequence_deleted`.
- `exit_on_reply` and `exit_on_unsubscribe` default to `true`: a contact who answers or opts out stops getting steps.
- `delete-sequence` ends live enrollments with `sequence_deleted`.
- `get-sequence` includes `stats`: `total_enrolled`, `active`, `paused`, `completed`, `exited`.

## Webhook events

`broadcast.scheduled`, `broadcast.sending`, `broadcast.sent`, `broadcast.failed`, `broadcast.cancelled`, `sequence.enrolled`, `sequence.step_sent`, `sequence.completed`, `sequence.exited`. Payloads are in the `socialit-webhooks` skill.
