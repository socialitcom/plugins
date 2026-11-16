# Socialit API v1 endpoints

Base URL `https://api.socialit.com/v1`. The main endpoints by area, with the matching MCP tool. Every endpoint, with parameters, schemas, and samples, is in the API reference at https://socialit.com/docs/api-reference/ and in the OpenAPI document at `https://api.socialit.com/v1/openapi.json` (each operation carries `x-access`, `x-mcp-tool`, `x-toolset`, and `x-idempotent`).

Permissions: `read` endpoints work with any key. `write` needs a `full` key. `admin` needs a `full` key that is not restricted to profiles, or, on OAuth, the `admin` scope and the Owner or Admin role.

## Credential and team

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /auth/verify` | `verify-credential` | Workspace, permission, and profiles of the key or token |
| `GET /api-keys`, `POST /api-keys` | `list-api-keys`, `create-api-key` | Admin. The secret is in the create response only |
| `PATCH /api-keys/{id}`, `DELETE /api-keys/{id}` | `update-api-key`, `delete-api-key` | Admin |
| `POST /api-keys/{id}/regenerate` | `regenerate-api-key` | Admin. The old secret stops working |
| `GET /users`, `GET /users/{id}` | `list-users`, `get-user` | Workspace members |
| `GET /logs`, `GET /activity` | `list-request-logs`, `list-activity` | Admin |

## Profiles, accounts, and connecting

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /profiles`, `POST /profiles` | `list-profiles`, `create-profile` | `external_id` holds your tenant ID. Create is admin |
| `GET /account-groups`, `POST /account-groups` | `list-account-groups`, `create-account-group` | Sets of accounts to target together |
| `GET /accounts` | `list-accounts` | Filter by `platform`, `profile_id`, `group_id`, `status` |
| `GET /accounts/health` | `get-accounts-health` | Stored health of every account; `live=true` probes tokens |
| `GET /accounts/{id}/health` | `get-account-health` | Live token and permission check |
| `GET /accounts/follower-stats` | `get-follower-stats` | Follower counts and growth |
| `PATCH /accounts/{id}` | `update-account` | Rename, or move to another profile |
| `DELETE /accounts/{id}` | `disconnect-account` | Revokes the token where supported |
| `GET /connect/platforms` | `list-connect-platforms` | Networks and their connect methods |
| `GET /connect/{platform}` | `get-connect-url` | Authorization URL. `profile_id`, `redirect_url`, `reconnect_account_id`, `scope_set`, `headless` |
| `POST /connect/{platform}` | `complete-connect` | Finish with `code` and `state` when your app receives the code |
| `GET /connect/pending-data`, `POST /connect/select` | `get-pending-connection`, `select-pending-connection` | Headless picker for Pages, organizations, locations, boards |
| `POST /connect/bluesky/credentials` | `connect-bluesky-credentials` | Handle and app password |
| `GET /connect/links`, `POST /connect/links` | `list-connect-links`, `create-connect-link` | Hosted, branded connect links (Business and Enterprise). Admin |

## Publishing

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /posts` | `list-posts` | Filter by `status`, `profile_id`, `account_id`, `platform`, `from` and `to` on `date_field` |
| `POST /posts` | `create-post` | Draft, schedule, queue, or publish now |
| `GET /posts/{id}`, `PATCH /posts/{id}`, `DELETE /posts/{id}` | `get-post`, `update-post`, `delete-post` | `delete_on_network=true` also deletes the live copies |
| `POST /posts/{id}/schedule` | `schedule-post` | Schedule, queue, or publish a draft |
| `POST /posts/{id}/cancel` | `cancel-post` | Stop a scheduled post |
| `POST /posts/{id}/retry` | `retry-post` | Re-publish failed postings |
| `POST /posts/{id}/edit`, `POST /posts/{id}/unpublish` | `edit-published-post`, `unpublish-post` | Change or delete the live post where the network allows |
| `POST /posts/{id}/duplicate` | `duplicate-post` | Copy as a draft |
| `POST /posts/{id}/approve`, `POST /posts/{id}/reject` | `approve-post`, `reject-post` | Workspace approvals |
| `POST /posts/bulk-upload` | `bulk-upload-posts` | CSV up to 5,000 rows; `dry_run=true` validates only |
| `GET /posts/platform-options` | `get-post-platform-options` | JSON schema of `platform_options` per network |
| `GET /postings`, `GET /postings/{id}` | `list-postings`, `get-posting` | Per-account results |
| `GET /publishing/settings`, `PATCH /publishing/settings` | `get-publishing-settings`, `update-publishing-settings` | Approvals and default time zone. Update is admin |
| `GET /queues`, `POST /queues` | `list-queues`, `create-queue` | Weekly slots per profile |
| `GET /queues/next-slot`, `GET /queues/preview` | `get-next-queue-slot`, `preview-queue` | Where the next posts land |
| `GET /templates`, `POST /templates` | `list-templates`, `create-template` | Reusable content |
| `POST /tools/validate/post` | `validate-post` | Same body as `POST /posts`; nothing is created |
| `POST /tools/validate/post-length` | `validate-post-length` | Character counts per network |
| `POST /tools/validate/media` | `validate-media-url` | Check a file against network limits |

## Media

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `POST /media/presign` | `presign-media-upload` | Returns `upload_url`, `headers`, and the media object |
| `POST /media/{id}/complete` | `complete-media-upload` | Finalize a presigned upload |
| `POST /media/upload-direct` | none | Multipart `file` upload |
| `POST /media` | `create-media-from-url` | Register a public URL |
| `GET /media`, `GET /media/{id}` | `list-media-library`, `get-media-item` | Library |
| `GET /media/usage` | `get-media-usage` | Storage against the plan |

## Inbox

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /inbox/conversations` | `list-conversations` | Direct messages |
| `GET /inbox/conversations/{id}/messages`, `POST` same path | `list-messages`, `send-message` | Sending reaches a real person |
| `GET /inbox/comments` | `list-comments` | Comments across accounts |
| `POST /inbox/comments/{id}/replies` | `reply-to-comment` | Public reply |
| `POST /inbox/comments/{id}/private-reply` | `send-private-reply` | Private reply to a commenter |
| `GET /inbox/reviews`, `POST /inbox/reviews/{id}/reply` | `list-reviews`, `reply-to-review` | Reviews |
| `GET /inbox/mentions`, `POST /inbox/mentions/reply` | `list-mentions`, `reply-to-mention` | Mentions |
| `GET /inbox/counts` | `get-inbox-counts` | Unread counters |

Details: the `socialit-inbox` skill.

## Analytics

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /analytics` | `list-post-analytics` | Metrics per post |
| `GET /analytics/posts/{postId}` | `get-post-analytics` | One post |
| `GET /analytics/delta` | `get-analytics-delta` | Changes since a cursor, instead of polling |
| `GET /analytics/dashboard`, `GET /analytics/daily-metrics` | `get-analytics-dashboard`, `get-daily-metrics` | Aggregates |
| `GET /analytics/best-time` | `get-best-time-to-post` | Best times to post |
| `GET /analytics/accounts/{accountId}` | `get-account-analytics` | One account |
| `GET /analytics/inbox/response-time` | `get-inbox-response-time` | Inbox analytics |

Details: the `socialit-analytics` skill.

## Contacts and automations

| Endpoint | Tool |
| --- | --- |
| `GET /contacts`, `POST /contacts` | `list-contacts`, `create-contact` |
| `GET /custom-fields`, `POST /custom-fields` | `list-custom-fields`, `create-custom-field` |
| `POST /broadcasts`, `POST /broadcasts/{id}/send`, `POST /broadcasts/{id}/schedule` | `create-broadcast`, `send-broadcast`, `schedule-broadcast` |
| `POST /sequences`, `POST /sequences/{id}/activate`, `POST /sequences/{id}/enroll` | `create-sequence`, `activate-sequence`, `enroll-sequence-contacts` |
| `POST /workflows`, `POST /workflows/{id}/activate`, `POST /workflows/{id}/trigger` | `create-workflow`, `activate-workflow`, `trigger-workflow` |
| `GET /comment-automations`, `POST /comment-automations` | `list-comment-automations`, `create-comment-automation` |

Details: the `socialit-automations` and `socialit-inbox` skills.

## Ads

| Endpoint | Tool |
| --- | --- |
| `GET /ads/connections`, `GET /ads/accounts` | `list-ad-connections`, `list-ad-accounts` |
| `GET /ads/campaigns`, `GET /ads/tree`, `GET /ads` | `list-ad-campaigns`, `get-ad-tree`, `list-ads` |
| `POST /ads/campaigns`, `POST /ads`, `POST /ads/boost` | `create-ad-campaign`, `create-ad`, `boost-ad-post` |
| `PATCH /ads/campaigns/{id}/status` | `set-ad-campaign-status` |
| `GET /ads/campaigns/{id}/analytics`, `GET /ads/insights` | `get-ad-campaign-analytics`, `get-ad-insights` |
| `GET /ads/leads`, `GET /ads/lead-forms` | `list-leads`, `list-lead-forms` |

Creating or resuming ads spends money. Details: the `socialit-ads` skill.

## Webhooks and events

| Endpoint | Tool |
| --- | --- |
| `GET /webhooks`, `POST /webhooks` | `list-webhooks`, `create-webhook` |
| `POST /webhooks/{id}/test`, `POST /webhooks/{id}/reset-secret` | `test-webhook`, `reset-webhook-secret` |
| `GET /webhooks/{id}/deliveries`, `POST /webhooks/deliveries/{id}/redeliver` | `list-webhook-deliveries`, `redeliver-webhook-delivery` |
| `GET /events/names`, `GET /events`, `GET /events/{id}` | `list-event-types`, `list-events`, `get-event` |

All webhook endpoints are admin. Details: the `socialit-webhooks` skill.

## Focus mode

| Endpoint | Tool |
| --- | --- |
| `GET /focus/platforms` | `list-focus-platforms` |
| `GET /accounts/{accountId}/focus` | `get-focus-manifest` |
| `GET`, `POST /accounts/{accountId}/focus/{resource}` | `list-focus-items`, `create-focus-item` |
| `GET`, `PATCH`, `DELETE /accounts/{accountId}/focus/{resource}/{itemId}` | `get-focus-item`, `update-focus-item`, `delete-focus-item` |
| `POST /accounts/{accountId}/focus/{resource}/{itemId}/{action}` | `run-focus-action` |

Typed routes per network, such as `GET /accounts/{accountId}/youtube/videos`, return the same shapes. Focus calls go to the network, so its quotas apply, and a resource without its scopes answers `403 missing_scopes` with a `reconnect_url`.

## Usage and billing

| Endpoint | Tool | Notes |
| --- | --- | --- |
| `GET /usage` | `get-usage` | Accounts against the allowance, the API request window, storage |
| `GET /usage/x-api`, `GET /billing/credits` | `get-x-api-usage`, `get-billing-credits` | X API usage and prepaid credits |
| `GET /billing`, `GET /billing/plans` | `get-billing`, `list-plans` | Plan, features, limits |

Checkout, plan changes, and payment methods are in the app only.
