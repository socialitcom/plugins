---
name: socialit-ads
description: This skill should be used when the user asks to "boost this post", "create a Facebook ad", "launch a campaign", "pause my ads", "change the ad budget", "how are my ads doing", "what did we spend this month", "build a custom audience", "create a lookalike", "download my leads", "set up a lead form", "install the pixel", or "send conversions" in Socialit. Covers ads connections and ad accounts, campaigns, ad sets, ads, boosts, creatives, targeting and reach estimates, audiences, insights, lead gen forms and leads, tracking tags, and conversions (toolsets `ads` and `ads_measurement`), with every change that can spend money created paused and confirmed first.
---

# Socialit ads

One ads model over Meta (Facebook and Instagram), Google Ads, LinkedIn, TikTok, Pinterest, X, Reddit, and OpenAI (ChatGPT) ads. A connection (`adc_`) reaches ad accounts (`ada_`), which hold campaigns (`adcmp_`), ad sets (`adset_`), and ads (`ad_`).

Ads need the `ads` feature, on the Business and Enterprise plans. Load the `socialit` skill first for workspaces, permissions, and errors.

## Ads spend real money

Every call that can start or raise spend needs the user's explicit yes on a written summary. Create everything paused, and never activate, raise a budget, or extend a schedule on your own initiative, even when the user asked for "the campaign" in general.

```text
Ad account:  Acme EU (ada_...), Meta, currency EUR, time zone Europe/Berlin
Objective:   traffic
Budget:      20.00 EUR per day (daily), at the ad set level
Schedule:    Oct 20 09:00 to Oct 27 23:59 Europe/Berlin (sent with the +02:00 / +01:00 offsets)
Targeting:   DE, AT; ages 25 to 54; interests: "Marketing software"
Creative:    image med_... headline "...", link https://...
Status:      created PAUSED; goes live only after you say "activate"
```

- Money is a decimal number in the **ad account's currency** (`50` is 50.00 EUR), never cents or micros. Read the currency and time zone from `get-ad-account` and write them in the summary.
- Send `start_time` and `end_time` as ISO 8601 with an offset or `Z`, so the time zone is never ambiguous.
- Budgets are `{ "amount": 20, "type": "daily" }` or `"lifetime"`. Always send the budget yourself; some networks fill in a default (TikTok gives a new ad set under a campaign without a budget 20 per day).
- Without an `end_time`, a campaign keeps running until someone pauses it. Set one whenever the user names an end date.

### Defaults to know

| Call | Default status |
| --- | --- |
| `create-ad`, `create-ad-campaign`, `create-ad-set`, `create-messaging-ad`, `create-call-ad` | Paused |
| `duplicate-ad-campaign`, `duplicate-ad-set`, `duplicate-ad` | Paused (`status_option`: `paused`, `active`, `inherited`) |
| `boost-ad-post` | **Active on every network.** Always pass `status: "paused"` unless the user approved going live in this call |

### Calls that can spend or raise spend

| Call | Why |
| --- | --- |
| `set-ad-campaign-status`, `set-ad-set-status`, `set-ad-status`, `bulk-set-ad-campaign-status` with `active` | Starts delivery |
| `boost-ad-post`, or any create or duplicate with `active` | Starts delivery |
| `update-ad-campaign`, `update-ad-set`, `update-ad` | Can change budget, bid, schedule, targeting, or status |
| `create-high-demand-period` | Schedules a budget increase (Meta) |
| `create-ad-shared-budget`, `create-ad-bid-strategy`, `update-ad-bid-strategy` | Budgets and bidding (Google) |
| `apply-ad-recommendations` | Google recommendations can raise budgets or add keywords; show each one first |
| `add-ad-keywords`, `update-ad-keyword` | New or re-enabled keywords on live campaigns |
| `reserve-ad-reach-frequency-prediction` | Reserves reach and frequency inventory |
| `update-ad-account-settings` | Can change the spend cap (Meta) |

Pausing (`status: "paused"`) never spends, so when the user asks to stop ads, do it right after confirming which ones. Deleting (`delete-ad-campaign`, `delete-ad-set`, `delete-ad`) cannot be undone; prefer pausing.

## Workflow

1. **Find the ad account.** `list-ad-connections`, then `list-ad-accounts`. Every ads call picks the account with `ad_account_id`, or `connection_id` plus `platform_ad_account_id` (such as `act_123`). If none is connected, `connect-ads` with a `platform` (`meta_ads`, `google_ads`, `linkedin_ads`, `tiktok_ads`, `pinterest_ads`, `x_ads`, `reddit_ads`) returns an authorization URL for the user to open; ChatGPT ads use `connect-openai-ads` with an API key.
2. **Read before writing.** `get-ad-account` (currency, time zone), `get-ad-account-finance` (balance, spend, spend cap on Meta), `get-ad-tree` for the existing campaign, ad set, and ad hierarchy.
3. **Plan the targeting.** `search-ad-targeting`, `search-ad-interests`, and `browse-ad-targeting` find targeting IDs; `estimate-ad-reach` sizes the audience; `get-ad-bid-pricing` and `get-ad-supply-forecast` suggest bids and delivery.
4. **Dry run.** `create-ad` with `validate_only: true` checks the whole tree without creating anything (Meta, Google, OpenAI). `preview-ad-creative` renders previews before anything exists.
5. **Confirm the summary**, then create it paused: `create-ad` builds campaign, ad set, creative, and ad in one call, or attaches to `existing_campaign_id` or `ad_set_id`.
6. **Review.** `get-ad-previews` and `get-ad-review` (the network's approval verdict). Show the user.
7. **Activate** only after a separate, explicit yes. With the default, the campaign, ad set, and ad were all created paused, so set each to `active`: `set-ad-campaign-status`, `set-ad-set-status`, `set-ad-status`.

Body fields, goals, targeting keys, creative fields, and examples are in `references/campaigns.md`.

### Boosting a post

`boost-ad-post` promotes a published post and keeps its engagement. Pass `post_id` (a Socialit `pst_` post published on that network) or `platform_post_id`, plus `name`, `goal`, `budget`, `end_time`, and `status: "paused"`. Meta boosts Facebook and Instagram posts, Google Ads boosts YouTube videos, and LinkedIn, TikTok, Pinterest, X, and Reddit boost their own posts. `list-ad-instagram-posts` lists Instagram posts that can be boosted.

## Reading performance

| Question | Tool |
| --- | --- |
| What's running and what did it cost | `list-ad-campaigns`, `list-ad-sets`, `list-ads` with `from_date`, `to_date`, `has_delivery`, `min_spend` |
| Daily spend for the account | `get-ad-timeline` |
| One campaign or ad, with breakdowns | `get-ad-campaign-analytics`, `get-ad-analytics` |
| Custom fields, breakdowns, attribution windows | `get-ad-insights`, or `create-ad-insights-report` then `get-ad-insights-report` for large reports |
| Search terms (Google) | `get-ad-search-terms` |
| Who changed what | `list-ad-account-activity` |

Synced lists refresh every 30 minutes; pass `live: true` to read straight from the network. After a new connection Socialit backfills 90 days, and synced reads answer `202` with `backfill_pending: true` until that finishes. Report spend with its currency, and cost per result (CPC, CPM, cost per lead) next to totals.

## Audiences, leads, and measurement

- **Audiences**: `list-ad-audiences`, `create-ad-audience` (`customer_list`, `website`, `lookalike`, `engagement`, `saved`, `company_list`), `add-ad-audience-users`. Plain emails and phone numbers are SHA-256 hashed by Socialit before they reach the network. Upload only customer data the user has the right to use for ads.
- **Lead gen**: `list-lead-forms`, `create-lead-form`, `list-leads`, `get-lead`, `list-lead-form-leads`. Leads are personal data: show only what the user asks for, and do not paste lead lists into unrelated places.
- **Tracking tags and conversions**: `list-tracking-tags`, `get-tracking-tag-diagnostics`, `send-conversions`, `get-event-match-quality`.

Details, including which calls are not idempotent and must not be retried blindly, are in `references/measurement.md`.

## Errors

| Error | Meaning |
| --- | --- |
| `409 capability_not_supported` | That network cannot do this operation; the error names `platform` and `capability` |
| `409 missing_scopes` | The connection lacks a permission; send the user to the `reconnect_url` |
| `202` with `backfill_pending: true` | The first sync is still running; try again later |
| `403` with `plan_required` | The workspace needs Business or Enterprise for ads |

The network's own rejection (policy, billing, invalid targeting) comes back with its message: quote it, fix the input, and confirm again before retrying a create. Ad events (`ad.status_changed`, `lead.received`, and more) are covered by the `socialit-webhooks` skill.

## Common requests

| Request | Tools |
| --- | --- |
| "Boost yesterday's Instagram post with 20 EUR a day for a week" | `list-ad-accounts`, `get-ad-account`, confirm, `boost-ad-post` with `status: "paused"`, then after a second yes set the campaign, ad set, and ad to `active` |
| "Pause everything on Meta" | `list-ad-campaigns` with `platform`, confirm the list, `bulk-set-ad-campaign-status` with `paused` |
| "How much did we spend last month?" | `get-ad-timeline` per ad account, totals in each currency |
| "Which ad has the cheapest leads?" | `list-ads` with the date range, sorted by spend, then `get-ad-analytics` |
| "Export this week's leads" | `list-leads` with `since` and `until` (add `is_test: false` to skip test leads) |
