# Socialit REST API v1 for one-off tasks

Use this when the MCP server is not connected and the user wants something done now with a key they already keep in `SOCIALIT_API_KEY`. For code that lives in the user's project, load the `socialit-api` skill instead; its `references/endpoints.md` maps every area.

Base URL: `https://api.socialit.com/v1`. Every request sends `Authorization: Bearer $SOCIALIT_API_KEY`. Bodies are JSON with snake_case fields. The OpenAPI 3.1 document is at `https://api.socialit.com/v1/openapi.json`.

## Tools and endpoints

Each MCP tool is one endpoint; the tool name is the operation ID in kebab case. The API reference at https://socialit.com/docs/api-reference/ shows the tool name on every endpoint.

| Tool | Endpoint |
| --- | --- |
| `verify-credential` | `GET /v1/auth/verify` |
| `list-accounts` | `GET /v1/accounts` |
| `get-accounts-health` | `GET /v1/accounts/health` |
| `get-connect-url` | `GET /v1/connect/{platform}` |
| `list-posts` | `GET /v1/posts` |
| `create-post` | `POST /v1/posts` |
| `get-post` | `GET /v1/posts/{id}` |
| `update-post` | `PATCH /v1/posts/{id}` |
| `schedule-post` | `POST /v1/posts/{id}/schedule` |
| `retry-post` | `POST /v1/posts/{id}/retry` |
| `validate-post` | `POST /v1/tools/validate/post` |
| `create-media-from-url` | `POST /v1/media` |
| `list-conversations` | `GET /v1/inbox/conversations` |
| `list-post-analytics` | `GET /v1/analytics` |
| `get-usage` | `GET /v1/usage` |

## Conventions

- Creates return `201` with the object. Updates use `PATCH` and return the full object. Deletes return `{ "object": "post", "id": "pst_...", "deleted": true }`. Actions are `POST /resource/{id}/{verb}`.
- Lists return one envelope. Follow `next_page_url` until `has_more` is `false`:

```json
{ "object": "list", "data": [], "page": 1, "limit": 20, "total": 123, "has_more": true, "next_page_url": "https://api.socialit.com/v1/posts?page=2&limit=20", "previous_page_url": null }
```

- Filters are query keys shaped `field.condition=value`, such as `content.contains=launch` or `scheduled_at.after=2026-10-01T00:00:00Z`. Add `match=or` to match any. Sort with `sort=<field>&order=asc|desc`.
- Errors share one shape. Branch on `type`, then `code`:

```json
{ "error": { "type": "invalid_request_error", "code": "validation_failed", "message": "The post cannot be published to X.", "param": "content", "errors": { "content": ["Too long for X (280)"] }, "platform": "x" }, "request_id": "req_..." }
```

- API keys are fixed to their workspace. OAuth tokens can pick another allowed workspace with the `X-Workspace` header (a `ws_` ID or the exact name).

## Common calls

Check the key and see its workspace and permission:

```bash
curl https://api.socialit.com/v1/auth/verify \
  -H "Authorization: Bearer $SOCIALIT_API_KEY"
```

Connected accounts and their health:

```bash
curl "https://api.socialit.com/v1/accounts?platform=linkedin" \
  -H "Authorization: Bearer $SOCIALIT_API_KEY"

curl https://api.socialit.com/v1/accounts/health \
  -H "Authorization: Bearer $SOCIALIT_API_KEY"
```

Create a draft (nothing is published). Show it to the user before scheduling:

```bash
curl -X POST https://api.socialit.com/v1/posts \
  -H "Authorization: Bearer $SOCIALIT_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: draft-webinar-2026-11-20" \
  -d '{ "content": "Join our webinar on Thursday.", "account_ids": ["sa_..."], "draft": true }'
```

After the user approves the post and the time, schedule it:

```bash
curl -X POST https://api.socialit.com/v1/posts/pst_.../schedule \
  -H "Authorization: Bearer $SOCIALIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "scheduled_at": "2026-11-20T09:00:00", "timezone": "America/New_York" }'
```

Posts scheduled this week:

```bash
curl "https://api.socialit.com/v1/posts?status=scheduled&from=2026-11-16T00:00:00Z&to=2026-11-23T00:00:00Z&sort=scheduled_at&order=asc" \
  -H "Authorization: Bearer $SOCIALIT_API_KEY"
```

On a failed post, `GET /v1/posts/{id}` returns one posting per account with `status`, `error`, `error_code`, and `error_category`.

## Limits

- The API and the MCP server share one hourly request budget per workspace: 600 on Free, 6,000 on Pro, 30,000 on Business, 120,000 or more on Enterprise.
- Responses carry `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` (seconds). A `429` adds `Retry-After`; wait that long before retrying.
- Send an `Idempotency-Key` header on creates so a retry does not publish twice.
