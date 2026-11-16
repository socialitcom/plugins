---
name: socialit-publishing
description: This skill should be used when the user asks to "schedule a post", "post this to LinkedIn and X", "publish now", "draft a post", "add this to my queue", "upload an image for a post", "why did my post fail", "retry a failed post", "edit a published post", "delete a post from Instagram", "approve a post", or "bulk upload posts from a CSV" in Socialit. Covers choosing target accounts, drafts, scheduling with time zones, queues, media uploads, per-network options, validation, approvals, postings and failures, editing and unpublishing, duplicates, templates, notes, and bulk upload, with explicit confirmation before anything goes live.
---

# Socialit publishing

A **post** (`pst_`) is what you write once. Every account it targets gets a **posting** (`sp_`) with its own status, network ID, URL, and error. A post to Instagram, LinkedIn, and X has three postings, and the post status rolls up from them.

Load the `socialit` skill first for workspaces, permissions, IDs, and errors.

## Nothing goes live without a yes

Publishing acts on real accounts in public. Default to a draft. Before any call that schedules, queues, publishes, or changes a live post, show the user a summary and wait for an explicit yes:

```text
Accounts: Acme on LinkedIn (sa_...), @acme on X (sa_...)
Text:     "We just shipped Focus mode." (X gets its own variant: "...")
Media:    launch.mp4 (med_...)
When:     Tue Oct 20, 9:00 AM Europe/Prague (07:00 UTC), or "now", or "next queue slot"
Options:  YouTube privacy public, made for kids: no
```

These calls put content on a network or change it there:

| Call | Effect |
| --- | --- |
| `create-post` with `publish_now`, `scheduled_at`, `queue`, or `queue_id` | Publishes now or later |
| `schedule-post` | Schedules, queues, or publishes an existing post (draft, cancelled, failed, or scheduled) |
| `approve-post` | Runs the requested action, and publishes at once if the scheduled time has passed |
| `retry-post` | Publishes the failed postings again |
| `update-post` on a scheduled post | Changes what goes out |
| `duplicate-post` with `scheduled_at`, `bulk-upload-posts` without `dry_run` | Creates scheduled posts |
| `edit-published-post`, `update-post-metadata` | Changes the live post on the network |
| `unpublish-post`, `delete-post` with `delete_on_network: true` | Deletes the live post on the network |

Traps to check every time:

- A `scheduled_at` in the past (or right now) publishes immediately. Confirm the time is in the future in the right time zone.
- `profile_id` without `account_ids` targets **every** account in the profile. `group_id` targets every account in the group. Narrow with `platforms`, and list the resulting accounts back to the user.
- `unpublish-post` without `account_id` or `platform` deletes the post on every account where it was published.
- `create-post` is marked as a create tool, so clients may not prompt before it publishes. The confirmation is your job.

## Workflow

1. **Pick the accounts.** `list-accounts` (filter by `platform`, `profile_id`, `group_id`, or `status`), `list-profiles`, `list-account-groups`. Accounts with `status: "needs_reconnect"` cannot publish; `get-accounts-health` shows why. Never guess an `sa_` ID.
2. **Prepare media** (below) and note each `med_` ID.
3. **Check network options.** `get-post-platform-options` returns the schema of `platform_options.<network>` for every network. Some networks need an explicit user choice: see `references/networks.md`.
4. **Validate.** `validate-post` takes the same body as `create-post`, creates nothing, and returns `errors` and `warnings` per account with length and limit. `validate-post-length` counts characters the way each network does (X counts URLs as 23 and emoji as 2). `validate-media-url` checks a file or `media_id` against each network.
5. **Create a draft.** `create-post` with `draft: true` (no schedule field also means draft). Draft warnings come back in `warnings`.
6. **Confirm**, then `schedule-post` with `scheduled_at` and `timezone`, `queue: true`, `queue_id`, or `publish_now: true`.
7. **Follow up.** Publishing is asynchronous. `get-post` shows the roll-up and every posting; `list-postings` filters by `post_id`, `account_id`, `platform`, or `status`.

## Targets

| Field | Targets |
| --- | --- |
| `account_ids` | These accounts |
| `profile_id` | Every account in the profile (filter with `platforms`) |
| `group_id` | Every account in the account group (filter with `platforms`) |
| `targets` | Per-account overrides: `account_id` plus `content`, `media_ids`, `platform_options`, `first_comment`, or `scheduled_at` |

`content` and media apply to every target unless a `targets` entry overrides them. Use overrides to fit X's 280 characters or to give LinkedIn a longer version.

## Scheduling and time zones

- Precedence when several are sent: `draft` > `publish_now` > `queue` / `queue_id` > `scheduled_at`.
- `scheduled_at` without an offset (`2026-10-20T09:00:00`) is wall clock time in `timezone` (an IANA name such as `America/New_York`). Without `timezone`, Socialit uses the default time zone in `get-publishing-settings`, then the workspace time zone, then UTC. Always send `timezone`, and ask the user which one they mean if it is unclear.
- Responses return `scheduled_at` in UTC. Convert back to the user's time zone when you report it.
- On an existing draft, `update-post` with `scheduled_at` keeps it a draft. Use `schedule-post` to schedule it.
- `cancel-post` stops a scheduled, pending approval, or publishing post. Postings already published stay live, and the post becomes `partially_published`.

### Queues

Queues are weekly slots (`day_of_week` 0 = Sunday, `time` as `HH:mm`) in the queue's time zone, per profile. `list-queues` and `get-queue` show them; `preview-queue` lists upcoming free slots and `get-next-queue-slot` the next one. Those two are informational: `queue: true` (the profile's default queue) or `queue_id` books the slot atomically when the post is created or scheduled. A queue with no free slot in the next 8 weeks answers `409 no_available_slot`. `create-queue` and `update-queue` change the slots; `reshuffle_existing: true` moves queued posts onto new slots, so confirm it.

## Media

Upload once and reuse the `med_` ID in any post.

- **Public URL**: `create-media-from-url` with `url` registers it in the library. `media_urls` on `create-post` does the same inline.
- **Local file**: `presign-media-upload` with `filename` and `content_type` returns `upload_url` (valid for one hour). PUT the bytes there with the returned `headers` (the same `Content-Type`) and no auth header, then call `complete-media-upload`. This needs an HTTP client or shell; a chat-only client cannot send file bytes, so ask for a public URL instead.
- **Library**: `list-media-library` (filter by `type`, `folder`, `tag`, `search`), `get-media-item`, `update-media-item` for alt text, caption, tags, and folder. `get-media-usage` shows storage against the plan.

Set alt text with `media: [{ "media_id": "med_...", "alt_text": "..." }]` on the post or with `update-media-item`. Videos get a thumbnail and preview after processing.

## Per-network options

Network-only settings go in `platform_options`, keyed by platform (`youtube`, `instagram`, `x`, ...). Keys may be camelCase or snake_case. Unknown options are dropped with a warning, or rejected with `422` when the post has `strict: true`. Other post-level fields:

- `first_comment`: a comment under the post on networks that support it.
- `thread_items`: continue the post as a thread on X, Threads, Bluesky, and Mastodon.
- `quote_post_id`, `quote_url`, `reply_to_url`: publish as a quote or reply.
- `recycle`: re-post evergreen content every `interval` `day`, `week`, or `month`, with optional `content_variations`. Not on YouTube or TikTok; up to 10 active recycling posts per account.

Per-network limits, required choices (TikTok privacy, YouTube made for kids), and helper tools are in `references/networks.md`.

## Approvals

With `require_approval` on (`get-publishing-settings`), scheduling by anyone who is not an owner, admin, or listed approver puts the post in `pending_approval`. Approvers call `approve-post` (optional `comment`), which runs the requested action, or `reject-post` (optional `comment` or `reason`), which returns it to `draft` and saves the comment as a note. Treat `approve-post` like publishing: confirm first. `403 approver_required` means the user cannot approve.

## After publishing

Statuses, error categories, what each status allows, and the common errors are in `references/post-lifecycle.md`.

- **Failed postings**: `list-postings` with `status: "failed"`, then read `error`, `error_code`, and `error_category`. Temporary network errors are retried automatically. Fix permanent causes (reconnect the account, shorten the text, change the media), then `retry-post`, optionally with `account_ids`. Only `failed` and `partially_published` posts can be retried.
- **Edit live**: `edit-published-post` with `content` (and `account_id` when several postings match). Not every network allows it; YouTube video details go through `update-post-metadata`.
- **Remove live**: `unpublish-post` with `account_id` or `platform` for one network. `delete-post` removes it from Socialit, and only touches the networks with `delete_on_network: true`.
- **Published posts** in Socialit accept only `title`, `tags`, `metadata`, and `recycle` through `update-post`.

## Reuse and collaboration

- `duplicate-post` copies content, media, and options into a new draft (or a scheduled post with `scheduled_at`), optionally for other `account_ids`.
- Templates (`list-templates`, `get-template`, `create-template`) store content, media, `platform_options`, and `first_comment`. Posts do not reference templates: read one and copy its fields into `create-post`.
- Internal notes: `list-post-notes`, `create-post-note`, or `note` on `create-post`. Notes never publish.
- `bulk-upload-posts` takes a CSV in `csv` (up to 5,000 rows). Run it with `dry_run: true` first, show the per-row results, and confirm before the real run.

## Common requests

| Request | Tools |
| --- | --- |
| "Post this to LinkedIn and X tomorrow at 9" | `list-accounts`, `validate-post`, `create-post` (draft), confirm, `schedule-post` |
| "Add it to the queue" | `get-next-queue-slot` to show the slot, confirm, `schedule-post` with `queue: true` |
| "What's going out this week?" | `list-posts` with `status: ["scheduled"]`, `from`, `to` |
| "Why did my Instagram post fail?" | `get-post`, `list-postings` with `status: "failed"`, `get-accounts-health` |
| "Fix the typo in my live post" | `get-post`, confirm, `edit-published-post` |
| "Take it down from X only" | confirm, `unpublish-post` with `platform: "x"` |
| "Approve everything pending" | `list-posts` with `status: ["pending_approval"]`, confirm each, `approve-post` |
