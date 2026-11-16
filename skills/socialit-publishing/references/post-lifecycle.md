# Post lifecycle, failures, and errors

## Post statuses

The post status is the roll-up of its postings.

| Status | Meaning |
| --- | --- |
| `draft` | Not scheduled |
| `pending_approval` | Waiting for an approver (workspace approvals on) |
| `scheduled` | Waiting for its time (a fixed time or a queue slot) |
| `publishing` | At least one posting is still running |
| `published` | Every posting published |
| `partially_published` | Some postings published, others failed or were cancelled |
| `failed` | No posting published |
| `cancelled` | Cancelled, or every published posting was unpublished |

## Posting statuses

Each target account has one posting (`sp_`).

| Status | Meaning |
| --- | --- |
| `pending` | Not published yet |
| `processing` | The network accepted it but has not given the final ID yet (Instagram and Threads containers, TikTok, Reddit media). It usually resolves on its own |
| `published` | Live, with `external_id` and `url` |
| `failed` | Not published; see `error`, `error_code`, `error_category` |
| `cancelled` | Cancelled before publishing, or unpublished later (`unpublished_at` is set) |

## What each status allows

| Action | Allowed when the post is |
| --- | --- |
| `update-post` (content, targets, media, schedule) | `draft`, `pending_approval`, `scheduled`, `failed`, `partially_published`, `cancelled` |
| `update-post` (only `title`, `tags`, `metadata`, `recycle`) | `published` |
| `schedule-post` | `draft`, `cancelled`, `failed`, `scheduled` |
| `cancel-post` | `scheduled`, `pending_approval`, `publishing` |
| `retry-post` | `failed`, `partially_published` |
| `approve-post`, `reject-post` | `pending_approval` |
| `edit-published-post`, `unpublish-post` | At least one posting is `published` |
| `delete-post` | Any status |

While a post is `publishing`, changes answer `409 publish_in_progress`. Wait and check `get-post` again. A wrong status answers `409 invalid_post_state`.

## Why a posting failed

`error_category` on a failed posting tells you what to do next. Socialit retries temporary failures by itself; permanent ones need a fix and then `retry-post`.

| `error_category` | Meaning | What to do |
| --- | --- | --- |
| `auth_expired` | The account's access was revoked or expired (`error_code` `account_needs_reconnect`) | Reconnect the account (see `socialit-get-started`), then retry |
| `account_issue` | Missing permissions, a feature the account cannot use (`missing_scopes`, `capability_not_supported`), a disconnected account, or a suspended workspace | Reconnect with the missing permissions, or remove that account |
| `user_content` | The content broke a network rule | Read `error`, fix the text, media, or options, validate, then retry |
| `platform_rejected` | The network refused the post | Read `error` and the network's rules; change the post before retrying |
| `platform_rate_limit` | The network rate limited the account | Retried automatically; if it still fails, retry later |
| `insufficient_credits` | Prepaid X credits ran out | The user buys credits, then retry |
| `quota_exhausted` | A network quota ran out (for example YouTube's daily quota) | Retry after the quota resets |
| `platform_error` | A temporary network error | Retried automatically; retry later if it still failed |
| `system_error` | A Socialit error | Retried automatically; retry once later, then contact support with the post ID |

Steps:

1. `get-post` to see every posting, or `list-postings` with `status: "failed"` to find failures across posts.
2. Explain the cause in plain words, quoting `error`.
3. For account problems, `get-accounts-health` or `get-account-health` shows token state and granted scopes.
4. After the fix (and the user's yes), `retry-post` with `account_ids` to retry only the fixed accounts.

## Errors when creating or scheduling

| Error | Meaning | What to do |
| --- | --- | --- |
| `422 validation_failed` | Field errors in `errors`, keyed like `content`, `media_ids[1]`, or `platform_options.x.replySettings` | Fix the fields; `validate-post` shows the same checks |
| `403 feature_unavailable` | A network has not approved Socialit's publishing yet | Remove those accounts, or keep the post as a draft (drafts only warn) |
| `403 account_publishing_paused` | The workspace has more connected accounts than the plan allows | Upgrade, buy extra accounts, or disconnect some |
| `402 insufficient_credits` | Publishing to X needs prepaid X credits | The user buys credits in Socialit |
| `429 posts_per_day_limit` | The plan's daily publish cap per account is reached | Schedule for the next UTC day |
| `409 no_available_slot` | The queue has no free slot in the next 8 weeks | Add slots or pick a time |
| `404 queue_not_found` | `queue: true` but the profile has no queue | Create one with `create-queue`, or schedule a time |
| `400 queue_inactive` | The queue is disabled | Activate it or use another |
| `409 nothing_to_retry` | No failed postings matched | Check `get-post` |
| `403 approver_required` | Only owners, admins, and approvers can approve or reject | Ask an approver |

`get-publishing-settings` shows the approval settings, the default time zone, `posts_per_account_per_day`, and the posts created this month.

## Webhooks

Post events (`post.scheduled`, `post.published`, `post.partially_published`, `post.failed`, `post.unpublished`, the approval events, and `posting.published` and `posting.failed` per account) let an app react without polling. Load the `socialit-webhooks` skill to set them up.
