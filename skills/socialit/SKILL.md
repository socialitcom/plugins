---
name: socialit
description: This skill should be used when the user asks to "post to Instagram with Socialit", "schedule a LinkedIn post", "list my Socialit accounts", "which Socialit workspace am I in", "switch to another Socialit workspace", "why did my post fail", "use the Socialit tools", or mentions Socialit, api.socialit.com, or the Socialit MCP server. Covers choosing between the Socialit MCP tools and the REST API v1, workspaces, roles and permissions, safe publishing with drafts and confirmation, toolsets, Focus mode, IDs, pagination, time zones, errors, and which Socialit skill to load for each task.
---

# Socialit

Socialit is API-first social media management. A workspace connects accounts on 14 networks, from Instagram and LinkedIn to Google Business Profile and Mastodon, and handles publishing, the inbox, analytics, contacts and automations, ads, and webhooks. Work with it through the hosted MCP server when it is connected, and through the REST API v1 when writing application code.

## Pick the interface

1. **MCP tools are available** (names like `get-current-workspace`, `list-accounts`, `create-post`): use them for anything the user wants done now, in their account.
2. **The user is writing code**: use the REST API at `https://api.socialit.com/v1`. Load the `socialit-api` skill.
3. **Neither**: offer to connect the MCP server (`socialit-get-started`). For a one-off task, the REST API works with a key the user keeps in `SOCIALIT_API_KEY` (see "Without MCP").

MCP tools run through the matching `/v1` endpoints, so validation, permissions, limits, and errors are the same.

Keys start with `sk_live_`. Read them from the environment or the client's secret store. Never put one in code, chat, or a commit, and never ask the user to paste one into the chat.

## First call

Call `get-current-workspace` before any other tool:

```json
{ "workspace": { "id": "ws_...", "name": "Northwind" }, "auth": "oauth", "role": "admin", "permissions": ["read", "write", "admin"], "can_switch_workspace": true }
```

Tell the user which workspace you are acting on before you change anything. `auth` is `oauth` or `api_key`; `role` is `null` for API keys.

## Multiple workspaces

An OAuth connection acts as the signed-in user and reaches the workspaces they allowed on the consent page: all of them, or a chosen set.

- `list-workspaces` returns the workspaces this connection may use, the user's role in each, and which one is current.
- When the user says "in Acme, do X", pass `workspace: "Acme"` (a `ws_` ID or the exact name) to that tool call. It applies to that call only. Do not switch first.
- `switch-workspace` (with `workspace_id`) changes the default for later calls. Use it only when the user wants to keep working there.
- A name shared by several workspaces returns `workspace_ambiguous`: use the ID. `workspace_not_allowed` or `workspace_not_found` means this connection does not include that workspace. The user reconnects Socialit in their client and selects it on the consent page.

API key connections are fixed to the key's workspace. They have no `workspace` argument, `list-workspaces`, or `switch-workspace`.

## Permissions and roles

Each tool needs one permission: `read`, `write` (posts, media, inbox replies, contacts, automations), or `admin` (profiles, API keys, webhooks, request logs, connect links, publishing settings).

| Role | Permissions |
| --- | --- |
| `owner`, `admin` | read, write, admin |
| `member` | read, write |
| `viewer` | read |
| `billing` | Billing and usage tools only (`get-usage`, `get-billing`, ...), which also allow `owner`, `admin`, and any API key |

- **OAuth**: the role, limited to the permissions the client was granted.
- **API keys**: `full` (read, write, admin) or `read`. A key restricted to profiles cannot use admin tools.
- **Profiles**: members, viewers, and restricted keys may see only some profiles. Accounts and posts outside them answer 404.

Tools the connection cannot call are hidden. On `admin_role_required`, `role_required`, or `insufficient_permission`, tell the user a workspace admin must do it, and do not retry. `insufficient_scope` means the client was not granted that permission: the user reconnects so it asks for it.

## Safety rules

You act in real accounts on real networks. Default to drafts, show exactly what will happen, and wait for a clear yes before any call that:

- **Publishes**: `create-post` with `publish_now`, `scheduled_at`, `queue`, or `queue_id`; `schedule-post`; `retry-post`; `approve-post`; `bulk-upload-posts` without `dry_run`; `edit-published-post`; `update-post-metadata`; `unpublish-post`. A `scheduled_at` in the past publishes at once.
- **Reaches people**: `send-message`, `reply-to-comment`, `create-comment`, `reply-to-review`, `reply-to-mention`, `send-private-reply`, `accept-conversation`, `send-broadcast`, `schedule-broadcast`, and automations that message people once running (`activate-sequence`, `enroll-sequence-contacts`, `activate-workflow`, `trigger-workflow`, `start-workflow-execution`, comment automations).
- **Acts on a network**: liking, hiding, pinning, or deleting comments, reposts and follows on X, Focus mode writes, and the Google Business and Discord tools in `platform_tools`.
- **Spends money**: creating or resuming ad campaigns, ad sets, ads, boosts, and budgets (load `socialit-ads`). Publishing to X and Focus calls on X use prepaid X credits.
- **Cannot be undone**: every `delete-*` tool, `disconnect-account`, `delete-profile`, `bulk-delete-contacts`, `cancel-post`, `cancel-broadcast`, `cancel-workflow-execution`, `regenerate-api-key`, `reset-webhook-secret`, `revoke-connect-link`. Name the object by its human name and ID.
- **Calls a URL**: `test-webhook`, `redeliver-webhook-delivery`, `retry-failed-webhook-deliveries`.

Tools carry MCP annotations from their kind (see `references/tools.md`), but many publishing and ad tools are kind `create` or `action`, so the client may not ask. You must.

- Create posts with `draft: true`, show the draft with `get-post`, and schedule only after the user approves (`schedule-post`).
- Run `validate-post` before scheduling to catch network limits.
- Never publish or send wording the user has not approved.

## Tools by area

| Area | Main tools | Skill |
| --- | --- | --- |
| Workspace and credential | `get-current-workspace`, `list-workspaces`, `switch-workspace`, `verify-credential` | this one |
| Accounts and connecting | `list-accounts`, `get-accounts-health`, `get-connect-url`, `list-profiles`, `list-account-groups` | `socialit-get-started` |
| Posts, queues, media | `create-post`, `schedule-post`, `list-posts`, `retry-post`, `presign-media-upload` | `socialit-publishing` |
| Inbox and comment automations | `list-conversations`, `send-message`, `reply-to-comment`, `list-reviews`, `create-comment-automation` | `socialit-inbox` |
| Analytics | `list-post-analytics`, `get-analytics-dashboard`, `get-best-time-to-post` | `socialit-analytics` |
| Contacts and automations | `list-contacts`, `create-broadcast`, `send-broadcast`, `create-sequence`, `create-workflow` | `socialit-automations` |
| Ads | `list-ad-campaigns`, `get-ad-campaign-analytics`, `create-ad-campaign`, `boost-ad-post` | `socialit-ads` |
| Webhooks and events | `create-webhook`, `list-events`, `list-webhook-deliveries` | `socialit-webhooks` |
| Focus mode | `list-focus-platforms`, `get-focus-manifest`, `list-focus-items`, `run-focus-action` | below |
| Team, keys, logs, billing | `list-users`, `create-api-key`, `list-request-logs`, `get-usage` | this one |

The full list with kind and permission for every tool is in `references/tools.md`.

## Toolsets and read only

The server has more than 670 tools in 25 toolsets. Clients can trim the list, so a tool may be missing:

- `?toolsets=posts,media,accounts` on the server URL, or the `X-MCP-Toolsets` header, keeps only those toolsets. `workspace` is always included.
- `?read_only=true`, or `X-MCP-Readonly: true`, keeps only read tools.
- A `read` key, or a connection without `write` or `admin`, hides the tools that need those permissions.

If a tool you need is missing, tell the user which toolset (the headings of `references/tools.md`) or permission to add instead of working around it.

## Focus mode

Focus mode covers what a network's API allows for one account beyond publishing and the inbox: YouTube playlists and captions, Reddit mod queues, Discord roles, Mastodon filters, and more.

- The generic `focus` toolset is always available. Call `get-focus-manifest` with an account ID: it lists the resources, their fields, operations, and actions, and whether each is `available`. Then call the item tools (`list-focus-items`, `create-focus-item`, `run-focus-action`, and so on) with the resource key.
- Typed toolsets per network (`focus_youtube`, `focus_reddit`, `focus_x`, and so on, about 2,000 tools) are opt in. The user adds them by name, for example `?toolsets=focus,focus_youtube`. Typed tools are named network, resource, operation, such as `youtube-videos-list`.
- A resource without its network permissions answers `403 missing_scopes` with a `reconnect_url`. Give the link to the user; reconnecting keeps the account and its history.
- Focus calls go straight to the network: its quotas apply and writes change the live account.

## Working with results

- IDs have prefixes: `ws_` workspace, `pro_` profile, `sa_` account, `grp_` account group, `pst_` post, `sp_` posting (one per target account), `med_` media, `que_` queue, `tem_` template, `conv_` conversation, `msg_` message, `cmt_` comment, `rev_` review, `mtn_` mention, `con_` contact, `brd_` broadcast, `seq_` sequence, `wf_` workflow, `wh_` webhook, `evt_` event, `key_` API key. Find IDs with list tools; never guess or shorten them.
- List tools take `page` (from 1) and `limit` (default 20, max 100), `sort`, `order` (`asc` or `desc`), and `filters: [{ field, condition, value }]`. Conditions: `exact`, `not_exact`, `contains`, `not_contains`, `starts_with`, `ends_with`, `empty`, `not_empty`, `gt`, `gte`, `lt`, `lte`, `before`, `after`. Filters combine with AND; `match: "or"` matches any. Many lists also take plain filters such as `status`, `platform`, and `profile_id`.
- Lists return `total` and `has_more`. Lists read live from a network (Focus resources, some ad data) use a cursor: pass `next_cursor` back as `cursor`.
- Platform keys are lowercase network names (`x`, `linkedin`, `google_business`, ...), as returned in each account's `platform`.
- Times come back in ISO 8601, UTC. Convert them to the user's time zone when you present them.
- When scheduling, send `scheduled_at` with an offset, or without one plus `timezone` (IANA, such as `America/New_York`). With neither, the workspace default time zone applies (`get-publishing-settings`). Confirm the time zone with the user.

## Errors

Tool errors include the message, field details, and a `Code` line.

| Status | Meaning | What to do |
| --- | --- | --- |
| 400 or 422 | Invalid input or `validation_failed`, such as too long for X | Read the field errors, fix the input, retry once |
| 401 | Sign in expired or revoked | Reconnect the MCP server |
| 402 | Prepaid X credits ran out | The user buys X credits in the app |
| 403 | Role, permission, profile access, or `plan_required` | Explain what is needed; do not retry |
| 403 `missing_scopes` | The account lacks network permissions | Share the `reconnect_url` |
| 404 | Not in this workspace, or in a profile you cannot see | Check the ID and the workspace; use a list tool |
| 409 | State conflict, such as `account_needs_reconnect` | Fetch the object and explain; reconnect the account if needed |
| 429 | Rate limit, `posts_per_day_limit`, or `platform_rate_limited` | Wait, then retry once; do not loop |
| 502 | The network refused (`platform_error`) | Report the network's message |
| 5xx | Socialit error | Retry later |

The API and the MCP server share one hourly request budget per workspace: 600 on Free, 6,000 on Pro, 30,000 on Business. Every tool call counts.

## Common tasks

For an area in the "Tools by area" table, load the skill named there. Also:

- **First time, or the tools are missing**: load `socialit-get-started`.
- **Which accounts need reconnecting?** `get-accounts-health`, then report accounts with `needs_reconnect` or status `error`.
- **Questions about Socialit itself**: the docs at https://socialit.com/docs/ (add `.md` to any page URL for Markdown), or `ask-support-question`, then `get-support-run` for the answer.

## Without MCP

Call `https://api.socialit.com/v1` with `Authorization: Bearer $SOCIALIT_API_KEY`. A tool name is its endpoint's operation in kebab case (`create-post` is `POST /v1/posts`). Envelopes, the endpoints behind common tools, and curl examples are in `references/rest-api.md`. For code in the user's project, load `socialit-api`.
