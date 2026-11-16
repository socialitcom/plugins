---
name: socialit-webhooks
description: This skill should be used when the user asks to "set up a Socialit webhook", "get notified when a post publishes", "send new DMs to my app", "verify the webhook signature", "handle post.failed events", "my webhook is failing", "redeliver a webhook", "rotate the webhook secret", or "which events can I subscribe to". Covers creating and updating endpoints, choosing events and profiles, the signing secret, verifying X-Socialit-Signature, retries and redelivery, delivery logs, the events API, and debugging failed deliveries.
---

# Socialit webhooks

Socialit sends a signed HTTPS `POST` to the user's endpoint when something happens in the workspace: a post publishes or fails, an account needs reconnecting, a DM or comment arrives, a contact changes, a broadcast or workflow runs, an ad or the billing changes. The tools are in the `webhooks` toolset.

Webhook tools need the `admin` permission: an OAuth connection of a workspace owner or admin that was granted `admin`, or a full API key that is not restricted to profiles. For a member the webhook tools are not listed and the API answers `admin_role_required`: tell them a workspace owner or admin must do it. The events API (`list-events`, `get-event`, `list-event-types`) needs only `read`. The number of endpoints depends on the plan (403 `webhook_limit_reached` when it is full).

## Create an endpoint

1. Ask for the endpoint URL and which events the app needs. Do not subscribe to everything by default; fewer events mean less load. The URL must be public HTTPS. Plain `http`, credentials in the URL, and private or internal addresses are refused.
2. Call `create-webhook` with `name`, `url`, and either `events: [...]` or `all_events: true` (every type, including types added later). Event names and payloads are in `references/events.md`.
3. Optional: `profile_ids` limits the endpoint to events about those profiles. Events that belong to no profile (billing, team members, the support assistant) only reach endpoints without `profile_ids`. `enabled` defaults to `true`. `secret` lets the user bring their own signing secret (16 to 256 characters).
4. The response includes `secret` (`whsec_...`). It is shown only here and after `reset-webhook-secret`; `get-webhook` never returns it. Tell the user to store it as `SOCIALIT_WEBHOOK_SECRET` in their app's secret store. Do not repeat it in chat more than needed, and never write it into code or a commit.
5. Offer `test-webhook`, and confirm first because it calls the user's server. It sends one signed request now and returns `success`, `status_code`, `response_body`, and `error`. Optional `type` picks the event type of the sample (default `webhook.test`).

Change an endpoint with `update-webhook`: `name`, `url`, `events`, `all_events`, `profile_ids` (`null` clears the filter), or `enabled`. Sending `events` replaces the whole list and turns `all_events` off unless you send it too. `delete-webhook` stops deliveries at once; confirm first.

## The request

```
POST <your url>
Content-Type: application/json
X-Socialit-Signature: <hex HMAC-SHA256>
X-Socialit-Timestamp: <unix seconds>
X-Socialit-Event: post.published
X-Socialit-Delivery: whd_...
```

The body is **one event**, a JSON object (not an array):

```json
{
  "id": "evt_...",
  "object": "event",
  "type": "post.published",
  "created_at": "2026-10-14T09:00:04.512Z",
  "workspace_id": "ws_...",
  "test": false,
  "data": { "post": { "object": "post", "id": "pst_...", "status": "published" } }
}
```

`data` depends on the type (`references/events.md`). Most payloads embed the full object in its API shape (a `post`, a `message`, an `account`); a few carry only IDs.

## Verify every request

Signature = hex of HMAC-SHA256, keyed with the whole webhook secret (including `whsec_`), over the string `"{timestamp}.{raw body}"`.

1. Read the **raw** body bytes before any JSON parsing. Re-serialized JSON does not match.
2. Compute the HMAC and compare it with `X-Socialit-Signature` in constant time.
3. Reject timestamps more than 300 seconds from now, to block replays. Every attempt is signed with a fresh timestamp, so retries still pass.
4. Return 401 on failure.

Code for Express, Next.js, Flask, Django, and PHP (with Laravel notes), plus a way to sign test requests, is in `references/verify-signature.md`.

## Handle events safely

- **Respond fast.** Return any 2xx, then do the work in a background job. An attempt times out after 15 seconds. Redirects are not followed, so a 3xx counts as a failure.
- **Deduplicate on the event `id`** (`evt_...`). Retries reuse the same `X-Socialit-Delivery`; a redelivery gets a new delivery ID but carries the same event `id`.
- **Order is not guaranteed.** Use `created_at` and the object's current state, not arrival order.
- **Skip test sends** in business logic: they have `test: true`. The `data` of a test is a placeholder sample, not the real payload shape, so build handlers against `references/events.md`.

## Retries and recovery

Any non-2xx response, timeout, or connection error is a failure. Socialit retries after 5 minutes, 30 minutes, 2 hours, and 5 hours, then every 12 hours, up to 11 attempts. Workspace owners and admins get an email after 5 failures in a row. An endpoint that keeps failing for 3 days is disabled (`enabled: false`, `disabled_reason: "failing_3_days"`), and deliveries to a disabled endpoint fail at once.

To recover, with the user's yes at each step (these tools call the user's server):

1. Fix the endpoint and check it with `test-webhook`. Test sends work on a disabled endpoint and never count as failures.
2. `update-webhook` with `enabled: true`. This also resets the failure streak.
3. `retry-failed-webhook-deliveries` queues every failed delivery of the last 3 days again and returns `retried` with the count. It does not re-enable the endpoint, so do step 2 first. It is limited to 10 calls per hour.
4. For a single event, `redeliver-webhook-delivery` with the delivery `id` queues a new delivery of the same event. Test deliveries cannot be redelivered.

## Delivery logs and the event log

- `list-webhook-deliveries` (one endpoint) and `list-all-webhook-deliveries` (every endpoint) filter by `status` (`pending`, `delivered`, `failed`), `event_type`, `event_id`, `webhook_id`, and `status_code`. Each delivery has `attempt`, `status_code`, `response_body` (first 2 KB), `error`, `duration_ms`, and `next_retry_at`. `get-webhook-delivery` returns one.
- `get-webhook` shows `enabled`, `failure_count`, `disabled_reason`, and `last_delivery_at`.
- `list-event-types` returns the catalog of types an endpoint can subscribe to.
- `list-events` (filter by `type`, `profile_id`, `created_at`) and `get-event` read the workspace event log: the same payloads, whether or not any endpoint received them. Use it to tell "the event never happened" apart from "the delivery failed".

## Debugging checklist

| Symptom | Check |
| --- | --- |
| Signature never matches | Raw body, the current secret (it changes on reset), the full secret with `whsec_`, and `timestamp + "." + body` |
| No requests at all | `enabled` is false, the type is not subscribed, `profile_ids` excludes the event, or the event never happened (`list-events`) |
| `status_code` 401 or 403 | Auth middleware, CSRF protection, or a firewall blocking the POST |
| `status_code` 301, 302, or 308 | Redirects are not followed; use the final URL |
| `error` mentions a timeout | Respond first, then process |
| `error` mentions a private or reserved address | The host resolves to an internal IP; use a public one |
| The same event handled twice | Deduplicate on the event `id` |
| Test passes but live events break the parser | Test `data` is a sample; use the real shapes |

## Rotate the secret

`reset-webhook-secret` returns a new `secret`, and the old one stops working at once: requests signed with the new secret fail verification until the app has it. Confirm first, then have the user update `SOCIALIT_WEBHOOK_SECRET` right away. Failed deliveries can be retried afterwards.

## Without MCP

REST endpoints under `https://api.socialit.com/v1`: `POST /webhooks`, `PATCH /webhooks/{id}`, `DELETE /webhooks/{id}`, `POST /webhooks/{id}/test`, `POST /webhooks/{id}/reset-secret`, `GET /webhooks/{id}/deliveries`, `POST /webhooks/{id}/retry-failed`, `POST /webhooks/deliveries/{id}/redeliver`, `GET /events`, `GET /events/names`. Authentication and request patterns are in the `socialit-api` skill.
