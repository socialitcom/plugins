# Audiences, leads, and measurement

These tools change no budgets, but several send personal data to the networks or create objects that cannot be safely retried. Confirm before calling them, and never retry a create that failed without checking whether it already exists.

## Custom audiences (`ads` toolset)

`create-ad-audience` with `name` and `type`:

| `type` | Built from | Key fields |
| --- | --- | --- |
| `customer_list` | Uploaded customers | `users` (up to 10,000 per call), `customer_file_source` |
| `website` | Visitors seen by a pixel | `pixel_id`, `rule`, `retention_days` (up to 540) |
| `lookalike` | People similar to a source audience | `source_audience_id`, `country`, `ratio` (0.01 to 0.2) |
| `engagement` | People who engaged with a Page or profile | `engagement_source` |
| `saved` | Saved targeting | `targeting` |
| `company_list` | Companies (LinkedIn) | Set the companies with `replace-ad-audience-companies` |

- `add-ad-audience-users` adds people (`email`, `phone`, `first_name`, `last_name`, `country`, `zip`, `external_id`, and more), up to 10,000 per call. Plain emails and phone numbers are SHA-256 hashed by Socialit before they reach the network. Send `hashed: true` on `create-ad-audience` when the data is already hashed.
- Upload only people the user has the right to use for advertising, and do not echo the uploaded personal data back in chat.
- `list-ad-audiences`, `get-ad-audience` (size and status), `update-ad-audience`, `delete-ad-audience` (confirm first).
- Use an audience in `create-ad` through `targeting.audience_ids`.

## Lead gen (`ads_measurement` toolset)

| Tool | Use |
| --- | --- |
| `list-lead-forms` | Meta instant forms of a Page, LinkedIn Lead Gen Forms of a sponsored account |
| `get-lead-form` | One form with its questions |
| `create-lead-form` | `name`, `questions` (1 to 30), and `privacy_policy_url` are required; `page_id` for Meta; `state`: `DRAFT` or `PUBLISHED` |
| `archive-lead-form` | Archive a form (confirm) |
| `list-leads` | Stored leads, filtered by `form_id`, `ad_id`, `campaign_id`, `since`, `until`, `is_test` |
| `get-lead` | One lead |
| `list-lead-form-leads` | Reads a form's leads live from the network and stores them; new ones fire `lead.received` |
| `create-test-lead`, `delete-test-leads` | Test the flow end to end |

Leads hold names, emails, and phone numbers. Show only what the user asked for. To route new leads into an app or CRM, the `socialit-webhooks` skill covers `lead.received`.

## Tracking tags (pixels)

- `list-tracking-tags` lists the pixels and measurement tags of an ad account; `get-tracking-tag` includes the install code. `list-ad-pixels` lists TikTok pixels with their optimization events.
- `get-tracking-tag-diagnostics` and `get-tracking-tag-stats` show whether a tag fires and what it receives. Start here when conversions look low.
- `create-tracking-tag` and `create-tracking-tag-event` are **not idempotent** on the networks. Check `list-tracking-tags` and `list-tracking-tag-events` before creating, and never retry blindly.
- `share-tracking-tag` and `unshare-tracking-tag` control which ad accounts can use a tag.
- Click URL parameters (UTMs), not pixels: `get-ad-tracking-params` and `set-ad-tracking-params`. On Meta, setting them rebuilds the ad's creative and repoints the ad.

## Conversions

| Tool | Use |
| --- | --- |
| `list-conversion-destinations` | Where the ad account can receive events (pixels, conversion actions, conversion rules); use its `id` as `destination_id` |
| `send-conversions` | Relay server-side events to the network's Conversions API |
| `adjust-conversions` | Retract, restate, or enhance conversions already sent (Google Ads) |
| `get-event-match-quality` | How well the sent events match people on the network |
| `list-conversion-actions`, `create-conversion-action` | Google Ads conversion actions with their tag snippets |
| `list-custom-conversions`, `create-custom-conversion` | Meta custom conversions |
| `create-conversion-destination` | A LinkedIn conversion rule (not idempotent) or a Google Ads conversion action (reused by name) |
| `get-conversion-metrics` | Attribution metrics for a destination |

`send-conversions` takes `destination_id` and up to 1,000 `events`, each with `event_name`, `event_time`, `event_id` (used to deduplicate), and `user`, plus optional `value`, `currency`, `items`, `order_id`, and `action_source`. Personal data is normalized and hashed, never stored by Socialit. Send a `test_code` first when the network supports test events, and pass `consent` (`ad_user_data`, `ad_personalization`) when the user has it. Events sent cannot be recalled except through `adjust-conversions` on Google Ads, so confirm the batch.

For code that sends conversions from a server, load the `socialit-api` skill.

## Product catalogs (Meta)

`list-catalogs`, `create-catalog`, `list-catalog-products`, `create-catalog-product`, `batch-catalog-products` (up to 5,000 changes, processed asynchronously; poll `get-catalog-batch`), feeds (`list-catalog-feeds`, `create-catalog-feed`), and product sets (`list-product-sets`, `create-product-set`). Deleting a catalog or product set breaks catalog ads that use it, so confirm.
