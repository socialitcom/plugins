---
name: socialit-api
description: This skill should be used when the user asks to "integrate Socialit", "call the Socialit API", "publish to social media from my app", "schedule posts from code with Socialit", "upload media to Socialit", "handle Socialit rate limits or errors", "let my users connect their social accounts", "add Socialit OAuth to my app", "switch from Zernio to Socialit", or is writing code that calls api.socialit.com. Covers the REST API v1 in application code: API keys and OAuth 2.1, verifying a key, the error envelope, pagination and filters, idempotency keys, rate limits, media uploads, creating and scheduling posts, connecting accounts, and generating a client from the OpenAPI document, with curl, Node, and Python examples.
---

# Socialit in application code

Write production code that calls the Socialit REST API v1. Detect the project's language from its files (`package.json`, `requirements.txt` or `pyproject.toml`, `composer.json`, `go.mod`, `Gemfile`) and follow its HTTP client and conventions.

Socialit has no official SDK. Use the language's standard HTTP client (`fetch` in Node 18+, `requests` or `httpx` in Python), or generate a typed client from the OpenAPI 3.1 document at `https://api.socialit.com/v1/openapi.json` with any OpenAPI generator. Do not invent a Socialit package.

## Rules

1. **Read the key from the environment**: `SOCIALIT_API_KEY`. Add it to `.env.example` with an empty value, keep `.env` in `.gitignore`, and never commit a key. Keys start with `sk_live_`.
2. **Call Socialit from a server**, never from browser or mobile code.
3. **Use the narrowest key**: `read` for code that only reads, `full` only for code that creates posts or changes data, and restrict the key to the profiles the integration serves. Keys are created in the app under **Manage > API keys** (workspace admins), or with `create-api-key` when the MCP server is connected.
4. **Verify the key at startup** with `GET /v1/auth/verify` and log the workspace it belongs to.
5. **Send an `Idempotency-Key`** on every create and send, derived from your own record ID, so a retry never publishes twice.
6. **Handle errors by `type`**: retry `429`, `502`, and `5xx` with backoff; do not retry other `4xx` unchanged.
7. **Prefer webhooks to polling** for publish results (`socialit-webhooks`).

## Basics

Base URL: `https://api.socialit.com/v1`. JSON bodies, snake_case fields, prefixed IDs (`sa_` account, `pst_` post, `med_` media, `pro_` profile, `ws_` workspace).

```bash
curl https://api.socialit.com/v1/auth/verify \
  -H "Authorization: Bearer $SOCIALIT_API_KEY"
```

```json
{ "object": "credential", "type": "api_key", "workspace": { "id": "ws_...", "name": "Northwind" }, "permission": "full", "scopes": ["read", "write", "admin"], "profile_ids": null }
```

- Creates return `201`. Updates use `PATCH` and return the full object. Deletes return `{ "object", "id", "deleted": true }`. Actions are `POST /resource/{id}/{verb}`.
- A `full` key gets read, write, and admin; a profile restricted `full` key gets read and write; a `read` key only reads. Keys can expire (`expires_at`) and be regenerated.
- API keys belong to one workspace. To act in several, use one key per workspace, or OAuth.

## Errors

```json
{ "error": { "type": "invalid_request_error", "code": "validation_failed", "message": "The post cannot be published to X.", "param": "content", "errors": { "content": ["Too long for X (280)"] }, "platform": "x" }, "request_id": "req_..." }
```

| `type` | Status | Handle |
| --- | --- | --- |
| `invalid_request_error` | 400, 422 | Fix the request; show `errors` per field |
| `authentication_error` | 401 | Missing, invalid, expired, or revoked key; do not retry |
| `permission_error` | 403 | Key permission or profile; `missing_scopes` carries a `reconnect_url` for the account |
| `not_found_error` | 404 | Wrong ID, or not visible to this key |
| `conflict_error` | 409 | State conflict, such as `account_needs_reconnect`, or an idempotent request still running |
| `plan_limit_error` | 402, 403 | `plan_required` (with `required_plan`), `account_limit_reached`, or prepaid X credits ran out |
| `rate_limit_error` | 429 | Wait `Retry-After` seconds |
| `platform_error` | 502 | The network refused; read `platform` and `platform_code` |
| `api_error` | 500 | Retry with backoff |

Log `request_id` with every failure; support asks for it. Do not parse `message`.

## Lists, filters, and pagination

Lists take `page` (from 1) and `limit` (default 20, max 100) and return `{ object: "list", data, page, limit, total, has_more, next_page_url, previous_page_url }`. Follow `next_page_url` until `has_more` is `false`. `total` can be `null` on very large lists.

Filter with `field.condition=value` (`exact`, `not_exact`, `contains`, `not_contains`, `starts_with`, `ends_with`, `empty`, `not_empty`, `gt`, `gte`, `lt`, `lte`, `before`, `after`), combined with AND unless `match=or`. Common shortcuts such as `status`, `platform`, and `profile_id` are plain query keys. Sort with `sort` and `order`. Lists read live from a network use `next_cursor`, passed back as `cursor`.

## Rate limits

The API and the MCP server share one hourly budget per workspace, across all keys and OAuth connections: 600 requests on Free, 6,000 on Pro, 30,000 on Business, 120,000 or more on Enterprise. Every response carries `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` (seconds). A `429` adds `Retry-After`. Some operations have tighter buckets, plans cap posts per account per day (`429 posts_per_day_limit`), and networks have their own limits (`429 platform_rate_limited`). `GET /v1/usage` shows the current window.

## Posts and media

`POST /v1/posts` targets `account_ids` (or a `profile_id` or `group_id`, filtered by `platforms`) with `content`, `media_ids` or `media_urls`, and `platform_options` per network. The schedule field decides what happens:

| You send | Result |
| --- | --- |
| `draft: true`, or no schedule field | Saved as a draft |
| `scheduled_at` (with an offset, or wall clock plus `timezone`) | Scheduled; a past time publishes now |
| `queue: true` or `queue_id` | Next free queue slot |
| `publish_now: true` | Published right away |

Precedence is `draft` > `publish_now` > `queue` > `scheduled_at`. Publishing is asynchronous: each target gets a posting (`sp_`) with its own status, URL, and error. Validate first with `POST /v1/tools/validate/post`; `POST /v1/posts` answers `422 validation_failed` with errors per field before anything is published. In development and tests, create drafts only.

Media is uploaded once and reused by ID: `POST /v1/media/presign`, `PUT` the file to `upload_url` with the returned `headers` (no auth header), then `POST /v1/media/{id}/complete`. Or register a public URL with `POST /v1/media`, or pass `media_urls` on the post.

Per-network options, queues, approvals, and failed posts are in the `socialit-publishing` skill.

## Patterns and endpoints

`references/patterns.md` has curl, Node, and Python code for:

- A small client with auth, timeouts, and typed errors.
- Retries with backoff, `Retry-After`, and idempotency keys.
- Paging through every result.
- Uploading media with presign, or from a URL.
- Creating a draft, validating, and scheduling.
- Connecting your users' accounts with connect links.

`references/endpoints.md` maps the main endpoints by area, with their MCP tool names.

## OAuth for apps that act for other users

Use OAuth 2.1 instead of asking users for API keys when your app acts for other Socialit users:

- Authorization code with PKCE (`S256`). Metadata: `https://api.socialit.com/.well-known/oauth-authorization-server`.
- Register once with dynamic client registration: `POST https://api.socialit.com/oauth/register` with `client_name` and `redirect_uris`. Public clients use `token_endpoint_auth_method: "none"`; confidential clients get a `client_secret` once.
- Send users to `https://api.socialit.com/oauth/authorize`, exchange the code at `https://api.socialit.com/oauth/token`, and revoke at `https://api.socialit.com/oauth/revoke`.
- Scopes: `read`, `write`, `admin`, and `offline_access`. Request only what the app needs. The user picks the workspaces on the consent page, and the token acts with their role.
- Access tokens last 15 minutes. Refresh tokens rotate: store the new one from every refresh.
- Send `X-Workspace: ws_...` to act in another workspace the user allowed. Users revoke apps under **Account > Connected apps**.

## Webhooks

Subscribe to `post.published`, `post.failed`, `posting.failed`, `account.needs_reconnect`, and inbox events instead of polling. Signing, verification, and retries are in the `socialit-webhooks` skill. Keep the signing secret (`whsec_...`) in `SOCIALIT_WEBHOOK_SECRET`.

## Migrating from Zernio

Socialit maps every Zernio endpoint. Most migrations are a base URL change, a new key, snake_case fields, and `PATCH` for updates. The full map is at https://socialit.com/docs/migrating-from-zernio/, and the OpenAPI document marks each operation's Zernio equivalent with `x-zernio`.
