# Verifying Socialit webhook signatures

Algorithm: `hex(HMAC_SHA256(key = webhook secret, message = timestamp + "." + raw_body))`, compared with the `X-Socialit-Signature` header.

- The key is the whole secret, including the `whsec_` prefix. Read it from `SOCIALIT_WEBHOOK_SECRET`.
- `timestamp` is the `X-Socialit-Timestamp` header exactly as sent (Unix seconds). Reject it when it is more than 300 seconds away from now.
- The header value is lowercase hex with no prefix.
- The body is one JSON event object, not an array. Deduplicate on its `id` (`evt_...`).
- Compare in constant time, and return 401 on any failure.

## Node.js (Express)

Mount the raw body parser on this route only, before any `express.json()` that would consume the body.

```js
import crypto from 'node:crypto';
import express from 'express';

const app = express();

function verifySocialit(rawBody, signature, timestamp, secret) {
  const ts = Number(timestamp);
  if (!signature || !Number.isFinite(ts) || Math.abs(Date.now() / 1000 - ts) > 300) return false;
  const expected = Buffer.from(crypto.createHmac('sha256', secret).update(`${timestamp}.${rawBody}`).digest('hex'));
  const given = Buffer.from(signature);
  return given.length === expected.length && crypto.timingSafeEqual(given, expected);
}

app.post('/webhooks/socialit', express.raw({ type: 'application/json' }), (req, res) => {
  const rawBody = req.body.toString('utf8');
  const valid = verifySocialit(
    rawBody,
    req.get('X-Socialit-Signature'),
    req.get('X-Socialit-Timestamp'),
    process.env.SOCIALIT_WEBHOOK_SECRET,
  );
  if (!valid) return res.sendStatus(401);

  const event = JSON.parse(rawBody);
  // Queue the work; the job ID deduplicates retries and redeliveries.
  queue.add('socialit-event', event, { jobId: event.id });
  res.sendStatus(200);
});
```

## Next.js route handler

```ts
import crypto from 'node:crypto';

export const runtime = 'nodejs';

export async function POST(request: Request) {
  const rawBody = await request.text();
  const signature = request.headers.get('x-socialit-signature') ?? '';
  const timestamp = request.headers.get('x-socialit-timestamp') ?? '';

  const ts = Number(timestamp);
  if (!signature || !Number.isFinite(ts) || Math.abs(Date.now() / 1000 - ts) > 300) {
    return new Response(null, { status: 401 });
  }
  const expected = Buffer.from(
    crypto.createHmac('sha256', process.env.SOCIALIT_WEBHOOK_SECRET!).update(`${timestamp}.${rawBody}`).digest('hex'),
  );
  const given = Buffer.from(signature);
  if (given.length !== expected.length || !crypto.timingSafeEqual(given, expected)) {
    return new Response(null, { status: 401 });
  }

  const event = JSON.parse(rawBody) as { id: string; type: string; test: boolean; data: Record<string, unknown> };
  // enqueue the event, deduplicating on event.id
  return new Response(null, { status: 200 });
}
```

## Python (Flask)

```python
import hashlib, hmac, json, os, time
from flask import Flask, abort, request

app = Flask(__name__)

def verify_socialit(raw_body: bytes, signature: str, timestamp: str, secret: str) -> bool:
    try:
        ts = int(timestamp)
    except (TypeError, ValueError):
        return False
    if not signature or abs(time.time() - ts) > 300:
        return False
    expected = hmac.new(secret.encode(), f"{timestamp}.".encode() + raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(signature.encode(), expected.encode())

@app.post("/webhooks/socialit")
def socialit_webhook():
    raw_body = request.get_data()  # bytes, before any JSON parsing
    if not verify_socialit(
        raw_body,
        request.headers.get("X-Socialit-Signature", ""),
        request.headers.get("X-Socialit-Timestamp", ""),
        os.environ["SOCIALIT_WEBHOOK_SECRET"],
    ):
        abort(401)
    event = json.loads(raw_body)
    handle_event.delay(event)  # deduplicate on event["id"]
    return "", 200
```

## Python (Django)

```python
import json, os
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from django.views.decorators.http import require_POST

@csrf_exempt
@require_POST
def socialit_webhook(request):
    raw_body = request.body  # bytes
    if not verify_socialit(  # the function from the Flask example
        raw_body,
        request.headers.get("X-Socialit-Signature", ""),
        request.headers.get("X-Socialit-Timestamp", ""),
        os.environ["SOCIALIT_WEBHOOK_SECRET"],
    ):
        return HttpResponse(status=401)
    event = json.loads(raw_body)
    # enqueue the event, deduplicating on event["id"]
    return HttpResponse(status=200)
```

## PHP

```php
<?php
$rawBody = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_SOCIALIT_SIGNATURE'] ?? '';
$timestamp = $_SERVER['HTTP_X_SOCIALIT_TIMESTAMP'] ?? '';

if ($signature === '' || !ctype_digit($timestamp) || abs(time() - (int) $timestamp) > 300) {
    http_response_code(401);
    exit;
}

$expected = hash_hmac('sha256', $timestamp . '.' . $rawBody, getenv('SOCIALIT_WEBHOOK_SECRET'));
if (!hash_equals($expected, $signature)) {
    http_response_code(401);
    exit;
}

$event = json_decode($rawBody, true);
// dispatch a job, deduplicating on $event['id']
http_response_code(200);
```

In Laravel, exclude the route from CSRF protection, read the body with `$request->getContent()`, and the headers with `$request->header('X-Socialit-Signature')` and `$request->header('X-Socialit-Timestamp')`.

## Testing a handler without Socialit

Sign a sample body yourself in a unit test (Node shown; any language works the same way):

```js
import crypto from 'node:crypto';

const secret = process.env.SOCIALIT_WEBHOOK_SECRET;
const body = JSON.stringify({ id: 'evt_test', object: 'event', type: 'post.published', created_at: new Date().toISOString(), workspace_id: 'ws_test', test: true, data: {} });
const timestamp = String(Math.floor(Date.now() / 1000));
const signature = crypto.createHmac('sha256', secret).update(`${timestamp}.${body}`).digest('hex');
// POST body with X-Socialit-Signature: signature and X-Socialit-Timestamp: timestamp
```

Then use `test-webhook` against the deployed URL to check the real path end to end.

## Common mistakes

| Mistake | Fix |
| --- | --- |
| A JSON body parser runs before verification | Verify the raw bytes, then parse |
| The secret is copied with a space or newline | Trim it when storing; the key is exactly `whsec_...` |
| The old secret after `reset-webhook-secret` | Update `SOCIALIT_WEBHOOK_SECRET` and redeploy |
| A proxy or framework rewrites the body | Read the body as it arrived, before any middleware changes it |
| Deduplicating on `X-Socialit-Delivery` | Use the event `id`; a redelivery has a new delivery ID |
| Answering after slow work | Return 2xx first; attempts time out after 15 seconds |
