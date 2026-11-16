# Socialit API patterns

Examples use Node 18+ (`fetch`) and Python 3 (`requests`). The same requests work from any language with a plain HTTP client. Every example reads the key from `SOCIALIT_API_KEY`.

## A small client

Node:

```js
const BASE = 'https://api.socialit.com/v1';

export class SocialitError extends Error {
  constructor(status, body, headers) {
    super(body?.error?.message ?? `Socialit request failed (${status})`);
    this.status = status;
    this.type = body?.error?.type;
    this.code = body?.error?.code;
    this.fields = body?.error?.errors;
    this.requestId = body?.request_id;
    this.headers = headers;
  }
}

export async function socialit(path, { method = 'GET', body, idempotencyKey, timeoutMs = 30_000 } = {}) {
  const headers = { Authorization: `Bearer ${process.env.SOCIALIT_API_KEY}` };
  if (body !== undefined) headers['Content-Type'] = 'application/json';
  if (idempotencyKey) headers['Idempotency-Key'] = idempotencyKey;
  const url = path.startsWith('https://') ? path : `${BASE}${path}`;
  const res = await fetch(url, {
    method,
    headers,
    body: body === undefined ? undefined : JSON.stringify(body),
    signal: AbortSignal.timeout(timeoutMs),
  });
  const text = await res.text();
  const data = text ? JSON.parse(text) : null;
  if (!res.ok) throw new SocialitError(res.status, data, res.headers);
  return data;
}
```

Python:

```python
import os
import requests

BASE = "https://api.socialit.com/v1"
session = requests.Session()
session.headers["Authorization"] = f"Bearer {os.environ['SOCIALIT_API_KEY']}"


class SocialitError(Exception):
    def __init__(self, status, body, headers):
        error = (body or {}).get("error", {})
        super().__init__(error.get("message", f"Socialit request failed ({status})"))
        self.status = status
        self.type = error.get("type")
        self.code = error.get("code")
        self.fields = error.get("errors")
        self.request_id = (body or {}).get("request_id")
        self.headers = headers


def socialit(method, path, json=None, idempotency_key=None, params=None):
    url = path if path.startswith("https://") else BASE + path
    headers = {"Idempotency-Key": idempotency_key} if idempotency_key else {}
    res = session.request(method, url, json=json, params=params, headers=headers, timeout=30)
    body = res.json() if res.content else None
    if not res.ok:
        raise SocialitError(res.status_code, body, res.headers)
    return body
```

Check the key at startup:

```js
const me = await socialit('/auth/verify');
console.log(`Socialit: ${me.workspace.name} (${me.workspace.id}), ${me.permission}`);
```

## Retries

Retry network errors, `429`, `502`, and `5xx` with backoff, and honor `Retry-After`. Keep the same `Idempotency-Key` on every attempt so a retried create returns the first result instead of creating twice. Do not retry other `4xx`, and do not retry `posts_per_day_limit` (the daily cap resets the next day).

```js
export async function withRetry(fn, { attempts = 4 } = {}) {
  for (let attempt = 1; ; attempt += 1) {
    try {
      return await fn();
    } catch (err) {
      const status = err instanceof SocialitError ? err.status : 0; // 0: network error or timeout
      const inProgress = status === 409 && err.code === 'idempotency_in_progress';
      const retryable = status === 0 || status === 429 || status >= 500 || inProgress;
      if (!retryable || err.code === 'posts_per_day_limit' || attempt >= attempts) throw err;
      const retryAfter = Number(err.headers?.get?.('retry-after'));
      const delay = retryAfter > 0 ? retryAfter * 1000 : 2 ** (attempt - 1) * 1000 + Math.random() * 250;
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}

const post = await withRetry(() =>
  socialit('/posts', { method: 'POST', body: draft, idempotencyKey: `campaign-${campaign.id}-post` }),
);
```

Idempotency rules:

- Keys are 1 to 255 characters of `A-Z a-z 0-9 _ - : .`, scoped to the workspace and the operation.
- The first `2xx` response is stored for 24 hours. A repeat with the same key and body returns it with `Idempotent-Replayed: true`.
- The same key with a different body answers `422 idempotency_key_reused`. Use a new key for a new request.
- Errors are not stored: fix the request and retry with the same key.

## Paging through a list

```js
export async function* listAll(path) {
  let next = `${path}${path.includes('?') ? '&' : '?'}limit=100`;
  while (next) {
    const page = await withRetry(() => socialit(next));
    yield* page.data;
    next = page.has_more ? page.next_page_url : null;
  }
}

for await (const account of listAll('/accounts?platform=instagram')) {
  console.log(account.id, account.username, account.status);
}
```

```python
def list_all(path, **params):
    params.setdefault("limit", 100)
    page = socialit("GET", path, params=params)
    while True:
        yield from page["data"]
        if not page.get("has_more"):
            return
        page = socialit("GET", page["next_page_url"])
```

Lists read live from a network return `next_cursor` instead of pages; pass it back as `cursor`. The `next_page_url` loop above works for both.

## Upload media

Presign, upload, complete:

```js
import { readFile, stat } from 'node:fs/promises';

const filePath = './launch.mp4';
const { size } = await stat(filePath);
const upload = await socialit('/media/presign', {
  method: 'POST',
  body: { filename: 'launch.mp4', content_type: 'video/mp4', size },
  idempotencyKey: `upload-${assetId}`,
});

// No Authorization header here: the URL is already signed.
const put = await fetch(upload.upload_url, {
  method: upload.method, // PUT
  headers: upload.headers, // includes Content-Type
  body: await readFile(filePath),
});
if (!put.ok) throw new Error(`Upload failed (${put.status})`);

await socialit(`/media/${upload.media.id}/complete`, { method: 'POST' });
const mediaId = upload.media.id; // med_...
```

```python
import os

path = "launch.jpg"
upload = socialit("POST", "/media/presign", json={
    "filename": "launch.jpg",
    "content_type": "image/jpeg",
    "size": os.path.getsize(path),
})
with open(path, "rb") as f:
    requests.put(upload["upload_url"], data=f, headers=upload["headers"], timeout=300).raise_for_status()
socialit("POST", f"/media/{upload['media']['id']}/complete")
```

The upload URL expires after `expires_in` seconds. For small files, `POST /v1/media/upload-direct` takes multipart `file` (images up to 20 MB, GIFs 15 MB, videos 500 MB, documents and audio 100 MB).

Register a file that is already public:

```bash
curl -X POST https://api.socialit.com/v1/media \
  -H "Authorization: Bearer $SOCIALIT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "url": "https://cdn.example.com/launch.jpg", "alt_text": "Product screenshot" }'
```

Or skip the library and pass `media_urls` on the post.

## Create, validate, schedule

```js
const draft = {
  content: 'Focus mode is live.',
  account_ids: ['sa_...', 'sa_...'],
  media_ids: [mediaId],
  platform_options: { youtube: { title: 'Focus mode walkthrough', privacyStatus: 'public', selfDeclaredMadeForKids: false } },
};

const check = await socialit('/tools/validate/post', { method: 'POST', body: draft });
if (!check.valid) {
  for (const target of check.targets) if (!target.valid) console.warn(target.platform, target.errors);
  throw new Error('Post would be rejected');
}

const post = await withRetry(() =>
  socialit('/posts', { method: 'POST', body: { ...draft, draft: true }, idempotencyKey: `launch-${launch.id}` }),
);

// Later, after review:
await withRetry(() =>
  socialit(`/posts/${post.id}/schedule`, {
    method: 'POST',
    body: { scheduled_at: '2026-11-20T09:00:00', timezone: 'Europe/Prague' },
    idempotencyKey: `launch-${launch.id}-schedule`,
  }),
);
```

- `scheduled_at` without an offset is wall clock in `timezone`; without `timezone` the workspace default applies. A time in the past publishes immediately.
- `GET /v1/posts/platform-options` returns the JSON schema of `platform_options` for every network. Some networks require a choice with no default (TikTok privacy level, YouTube made for kids).
- The post's `status` rolls up its postings: `draft`, `pending_approval`, `scheduled`, `publishing`, `published`, `partially_published`, `failed`, `cancelled`. Each posting has `status`, `url`, `error`, `error_code`, and `error_category`.
- Retry failed postings with `POST /v1/posts/{id}/retry` (optionally `account_ids`) after fixing the cause.
- Learn the outcome from the `post.published`, `post.partially_published`, and `post.failed` webhooks, or poll `GET /v1/posts/{id}` with backoff.

## Connect your users' accounts

For a product where each of your customers connects their own social accounts:

1. Create one profile per customer, keyed by your tenant ID:

```js
const profile = await socialit('/profiles', {
  method: 'POST',
  body: { name: customer.name, external_id: customer.id },
  idempotencyKey: `profile-${customer.id}`,
});
```

2. Get a connect URL and redirect the user's browser to `url`:

```js
const params = new URLSearchParams({ profile_id: profile.id, redirect_url: 'https://your-app.com/socialit/connected' });
const { url } = await socialit(`/connect/linkedin?${params}`);
```

3. The user comes back to `redirect_url` with `connected`, `account_ids`, and `profile_id`, or with `error` and `error_message` (`oauth_denied`, `account_limit_reached`, and others). Also listen for the `account.connected` webhook.

`GET /v1/connect/platforms` lists every network and its connect method. Bluesky connects with an app password (`POST /v1/connect/bluesky/credentials`), and Mastodon needs `instance`. On Business and Enterprise, `POST /v1/connect/links` creates a branded hosted connect link instead. Reconnect an account in place with `reconnect_account_id=sa_...`; reconnects never count against the plan's account allowance.

Use a key restricted to that customer's profile when a job works for one customer only.

## Testing

- Unit tests: mock the HTTP client and assert on the request body and headers you send.
- Integration tests: a separate workspace and a profile restricted key, and drafts only (`draft: true`). Never point automated tests at real customer accounts.
- Start new integrations with a `read` key, then switch to `full` for writes.
