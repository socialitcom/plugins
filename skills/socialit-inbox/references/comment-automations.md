# Comment automations (comment-to-DM)

A comment automation watches comments on one account, or on one post, and answers each matching comment at most once: a DM (private reply) and a public reply. Tools: `list-comment-automations`, `create-comment-automation`, `get-comment-automation`, `update-comment-automation`, `delete-comment-automation`, `list-comment-automation-logs`. They need the `automations` plan feature (Pro and above).

## Networks

| Networks | What it sends | Rules |
| --- | --- | --- |
| Instagram, Facebook | DM and public reply | `dm_message` (or `template`) is required |
| TikTok, Threads, LinkedIn, YouTube, Bluesky, Reddit | Public reply only | `comment_reply` is required; DM fields are refused with 422 `dm_not_supported` |
| X and others | Not available | 422 `platform_not_supported` |

The `live_comment`, `story_reply`, and `story_mention` triggers, `audience`, and `follow_gate` are Instagram only.

## Fields

| Field | Notes |
| --- | --- |
| `account_id`, `name` | Required |
| `trigger` | `comment` (default), `live_comment`, `story_reply`, `story_mention` |
| `post_id` | A Socialit post (`pst_...`). The automation arms itself when that post publishes, so it can be set up before a scheduled post goes live |
| `platform_post_id` | A network media ID instead (or a story ID for `story_reply`). Omit both for the whole account |
| `keywords` | Up to 50. **Empty means every comment** |
| `match_mode` | `contains` (default), `word`, or `exact`; case insensitive |
| `exclude_keywords` | Comments containing any of these are ignored |
| `typo_tolerance` | With `word`: 1 typo allowed for 4 to 7 letters, 2 for 8 or more |
| `dm_message` | Up to 1,000 characters, or 640 with buttons |
| `dm_message_variations`, `comment_reply_variations` | Up to 5 alternatives, picked at random |
| `buttons` | Up to 3: `{ "type": "url" or "postback", "title" (up to 20 characters), "url" or "payload" }`. Not combined with `template` |
| `template` | A card carousel: `{ "type": "generic", "elements": [{ "title", "subtitle", "image_url", "buttons" }] }`, 1 to 10 elements |
| `quick_replies` | Up to 13, not combined with buttons or a template |
| `dm_media` | `{ "type": "image", "url": "..." }`, sent as a second message |
| `comment_reply` | The public reply |
| `public_reply_policy` | `after_dm` (default) or `always` |
| `dm_delay_seconds`, `comment_reply_delay_seconds` | 0 to 86,400. The public reply never comes before the DM |
| `link_tracking` | Default `true`: URL buttons go through a tracked link |
| `click_tag` | Tag added to the contact when they click a tracked link |
| `also_match_in_dms` | The keywords also fire on DMs (needs at least one keyword; not with `story_reply`) |
| `audience` | Instagram: `follower_status` (`any`, `follower`, `non_follower`), `min_follower_count`, `when_unknown` (`send`, `skip`, `verify`), `tap_to_unlock` |
| `follow_gate` | Instagram: ask the person to follow before the DM: `message`, `button_label`, `not_following_message` |
| `repeat_policy` | `{ "mode": "once" }` (default) or `{ "mode": "every_comment", "cooldown_hours": 24 }` |
| `dedupe_same_text_hours` | Skip a commenter who posts the same text again within this many hours |
| `actions` | `{ "like_comment": true, "hide_comment": false }` |
| `is_active` | **Default `true`.** Create with `false`, then activate after the user approves |

Personalize the DM, the reply, and button titles with `{{first_name}}`, `{{name}}`, and `{{username}}`.

## Example

```json
{
  "account_id": "sa_...",
  "name": "Price list",
  "post_id": "pst_...",
  "keywords": ["price", "prices"],
  "match_mode": "word",
  "dm_message": "Hi {{first_name}}, here is our price list.",
  "buttons": [{ "type": "url", "title": "Open prices", "url": "https://example.com/prices" }],
  "comment_reply": "Sent you a DM!",
  "repeat_policy": { "mode": "once" },
  "is_active": false
}
```

Before turning it on, tell the user in plain words: which post or account it watches, which comments match, the exact DM and public reply, and that it will send them automatically to everyone who matches.

## Which automation answers

- Each comment is answered at most once per automation.
- When several automations match, the first one wins, with post-specific automations before account-wide ones.
- A matching comment automation answers before any workflow with a `comment` trigger.
- Messages sent by automations never start workflows.
- The private reply follows Meta's rules: one per comment, within 7 days of the comment.

## Results

- `get-comment-automation` returns `stats`: `triggered`, `dms_sent`, `dms_failed`, `unique_contacts`, `public_replies_sent`, `link_clicks`, `unique_clicks`, `audience_skipped`, and follow gate counters.
- `list-comment-automation-logs` lists every trigger with the commenter, the comment text, `status` (`pending`, `sent`, `failed`, `skipped`, `gated`), `skip_reason`, `error`, and `comment_reply_status`. Filter by `status` and `source`. Logs are kept 90 days.
- To pause an automation, `update-comment-automation` with `is_active: false`. `delete-comment-automation` removes it; confirm first.
