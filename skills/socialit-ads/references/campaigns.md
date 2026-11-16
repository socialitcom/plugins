# Campaigns, ad sets, and ads

Not every network supports every field or operation. An unsupported call answers `409 capability_not_supported` with `platform` and `capability`, so try the call (paused or with `validate_only`) rather than assuming.

## Selecting the ad account

Every call takes one of:

- `ad_account_id`: the Socialit ad account (`ada_`), from `list-ad-accounts`.
- `connection_id` (`adc_`) plus `platform_ad_account_id`: the network's own ID (`act_123` on Meta, the Google customer ID, the TikTok advertiser ID).

`update-ad-account` with `selected` chooses which ad accounts Socialit manages, syncs, and bills; confirm before selecting more. `create-ad-account` creates a Meta ad account in a business portfolio, and the user must then add a payment method in Meta's Ads Manager.

## `create-ad`: the whole tree in one call

| Field | Meaning |
| --- | --- |
| `name` (required) | Base name; `campaign_name`, `ad_set_name`, `ad_name` override each level |
| `goal` | `traffic`, `engagement`, `awareness`, `video_views`, `lead_generation`, `lead_conversion`, `conversions`, `app_promotion`, `catalog_sales`, `page_likes`, `page_visits`, `job_applicants`. Mapped to each network's objective |
| `status` | `paused` (default) or `active`. `campaign_status`, `ad_set_status`, `ad_status` set one level |
| `budget` | `{ "amount": 20, "type": "daily" }` or `"lifetime"`, in the ad account's currency |
| `budget_level` | `campaign` or `ad_set` |
| `start_time`, `end_time` | ISO 8601. Include an offset or `Z` so the time zone is explicit |
| `bid_strategy`, `bid_amount`, `optimization_goal`, `billing_event` | Bidding, in the network's terms |
| `targeting` | Normalized targeting (below), or `{ "raw": { ... } }` with the network's own spec |
| `creative` or `creatives` (up to 50) | Creative fields (below) |
| `existing_campaign_id`, `ad_set_id` | Attach to an existing campaign or ad set instead of creating one |
| Identity | `page_id` and `instagram_account_id` (Meta), `identity_id` and `identity_type` (TikTok, see `list-tiktok-ad-identities`), `organization_id` (LinkedIn), `board_id` (Pinterest) |
| `special_ad_categories` | Meta's restricted categories, when the ad falls in one |
| `lead_gen_form_id` | Lead ads: the form from `list-lead-forms` |
| `campaign_type` | Google: `search`, `display`, `performance_max`, `demand_gen`, `video` (with `keywords`, `headlines`, `descriptions`, `business_name`) |
| `validate_only` | `true` checks everything and creates nothing (Meta, Google, OpenAI) |

Example, as a dry run first:

```json
{
  "ad_account_id": "ada_5Rt8Yp1Xn3Vb8Kd5SzHc2Lm9Qw4",
  "name": "Focus mode launch",
  "goal": "traffic",
  "status": "paused",
  "budget": { "amount": 20, "type": "daily" },
  "budget_level": "ad_set",
  "start_time": "2026-10-20T09:00:00+02:00",
  "end_time": "2026-10-27T23:59:00+01:00",
  "targeting": { "countries": ["DE", "AT"], "age_min": 25, "age_max": 54 },
  "page_id": "1234567890",
  "creative": {
    "headline": "Focus mode is live",
    "body": "Everything a network allows, in one place.",
    "link_url": "https://example.com/focus",
    "image_url": "https://example.com/launch.png"
  },
  "validate_only": true
}
```

Remove `validate_only` for the real call only after the user confirms the summary. The response has the created `campaign`, `ad_set`, `ads`, and any `warnings`.

### Targeting keys

`countries` (ISO codes), `regions`, `cities`, `zips`, `excluded_locations`, `age_min`, `age_max`, `genders` (`male`, `female`), `languages`, `interests`, `behaviors`, `job_titles`, `industries`, `audience_ids` (Socialit `adaud_` IDs work in `create-ad`) and `excluded_audience_ids`, `placements`, `keywords`, `subreddits`. Find IDs for interests, regions, and the like with `search-ad-targeting`, `search-ad-interests`, or `browse-ad-targeting`, then size the audience with `estimate-ad-reach`.

Always set a location. On Meta, Pinterest, and Reddit, targeting without one defaults to the United States.

### Creative fields

`headline`, `body`, `description`, `call_to_action` (the network's own value), `link_url`, `image_url` or `image_hash`, `video` (`{ "url": ... }` or `{ "id": ... }`), `carousel_cards`, `existing_creative_id`, `url_tags`. Upload assets first with `upload-ad-image` or `upload-ad-video` (check processing with `get-ad-video`), or reuse one from `list-ad-creatives`. `preview-ad-creative` renders previews before creating; `get-ad-previews` and `get-ad-review` show the live ad and the network's review verdict after.

## Other ways to create

| Tool | Use | Default status |
| --- | --- | --- |
| `boost-ad-post` | Promote a published post (`post_id` or `platform_post_id`, `name` and `goal` required) | **Active** |
| `create-messaging-ad` | Click to message (`destination`: `messenger` or `instagram_direct`, optional `welcome_message`) | Paused |
| `create-call-ad` | Click to call (`phone_number` required) | Paused |
| `create-ad-campaign`, `create-ad-set` | One level at a time | Paused |
| `duplicate-ad-campaign`, `duplicate-ad-set`, `duplicate-ad` | Copy (`deep_copy`, `status_option`, new `start_time` and `end_time`, rename options) | Paused |

## Changing live objects

- Pause or resume: `set-ad-campaign-status`, `set-ad-set-status`, `set-ad-status` with `status`; `bulk-set-ad-campaign-status` takes up to 50 campaigns and reports `updated`, `skipped`, and `failed`.
- Edit: `update-ad-campaign` (name, budget, bidding, schedule, `spend_cap`), `update-ad-set` (budget, bidding, targeting, schedule, status), `update-ad` (name, status, creative, and on some networks budget and targeting). Before changing a budget, read the current one (`get-ad-campaign`, `get-ad-set`, or `get-ad-account-live` on Meta and TikTok) and show old and new side by side.
- Delete: `delete-ad-campaign`, `delete-ad-set`, `delete-ad`. Irreversible; offer pausing instead.

## Reading performance

- Synced lists (`list-ad-campaigns`, `list-ad-sets`, `list-ads`) take `from_date`, `to_date`, `status`, `has_delivery`, `min_spend`, `search`, and sort by `spend` or `impressions`. `live: true` reads through to the network.
- `get-ad-tree` shows the hierarchy; `get-ad-timeline` daily metrics per ad account.
- `get-ad-campaign-analytics` and `get-ad-analytics` return a `summary`, a `daily` series, and `breakdowns` (with `unsupported_breakdowns` when a network lacks one).
- `get-ad-insights` is a flexible live query (`level`, `fields`, `breakdowns`, attribution windows, `time_increment`). For big reports use `create-ad-insights-report`, then poll `get-ad-insights-report`.

## Google Ads specifics

- Keywords: `list-ad-keywords`, `add-ad-keywords`, `update-ad-keyword` (pause or enable), `delete-ad-keyword`; negative keywords per campaign or in shared lists.
- Planning: `generate-ad-keyword-ideas`, `get-ad-keyword-historical-metrics`, `get-ad-search-terms`.
- Recommendations: `list-ad-recommendations`, then `apply-ad-recommendations` or `dismiss-ad-recommendations`. Show each recommendation's effect before applying.
- Bidding and budgets: `list-ad-bid-strategies`, `create-ad-bid-strategy`, `list-ad-shared-budgets`, `create-ad-shared-budget`.
- Assets: sitelinks, callouts, and structured snippets per account, campaign, or ad group; Performance Max asset groups (`list-ad-asset-groups`, `create-ad-asset-group`).
- Local Services: `list-local-services-leads`.

## Other tools

- Ad comments: `list-ad-comments`, `reply-ad-comment` (public, confirm the text), `hide-ad-comment`, `delete-ad-comment`.
- Research: `search-ad-library` (the public Ad Library), `list-ad-studies` (A/B tests and lift studies).
- Reach and frequency (Meta): `create-ad-reach-frequency-prediction` and `get-ad-reach-frequency-prediction` are planning; `reserve-ad-reach-frequency-prediction` commits inventory, so confirm it like a budget.
- EU ads: `get-ad-account-dsa-recommendations` suggests the beneficiary and payor names for DSA disclosures; nothing is applied automatically.
