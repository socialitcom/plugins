---
name: socialit-analytics
description: This skill should be used when the user asks "how did my posts do", "what's my best time to post", "show follower growth", "which post performed best last month", "compare Instagram and LinkedIn", "how fast do my posts fade", "does posting more help", "show audience demographics", "get YouTube channel insights", "how fast do we answer DMs", or "build a monthly social report" in Socialit. Covers post analytics, account analytics and demographics, follower stats, best time to post, posting frequency, content decay, daily metrics, the analytics dashboard, network-specific insights (YouTube, LinkedIn, TikTok, Facebook, Instagram, Google Business Profile), inbox analytics, syncing, date ranges and time zones, and how to present results.
---

# Socialit analytics

Socialit stores daily snapshots of account and post metrics, so history survives even when a network only returns recent data. Every tool here is read only except the two sync tools. Answer with numbers from the account, name the date range and accounts, and say what the data cannot show.

Load the `socialit` skill first for workspaces, permissions, and errors.

## Conventions

- `from_date` and `to_date` are UTC days (`YYYY-MM-DD`), inclusive. Convert the user's "last week" or "October" into UTC days, and say which days you used.
- `account_id`, `profile_id`, and `platform` filter the aggregates. You only see accounts you can access.
- `source`: `all` (default), `socialit` (published through Socialit), or `external` (posted natively on the network and imported).
- Metrics are normalized: `impressions`, `reach`, `views`, `likes`, `comments`, `shares`, `saves`, `clicks`. A metric a network does not report is `null`, never `0`. Never treat `null` as zero.
- `engagement_rate` is a percentage: engagements divided by impressions (else reach, else views), times 100. It is `null` for YouTube.

### History by plan

| Plan | History |
| --- | --- |
| Free | 30 days |
| Pro | 1 year |
| Business | 2 years |
| Enterprise | Unlimited |

An older `from_date` answers `403 analytics_history_limit` with the earliest allowed day. Suggest a shorter range or the plan that covers it.

## Pick the tool

| Question | Tool |
| --- | --- |
| Overall performance for a period, with top posts | `get-analytics-dashboard` (needs `from_date` and `to_date`; `compare: "previous_period"` adds the prior period) |
| Which posts did best | `list-post-analytics` with `sort` (`engagement_rate`, `engagement`, `impressions`, `views`, ...) and `order` |
| One post's numbers | `get-post-analytics` (accepts `pst_`, `sp_`, or the network's post ID) |
| How one post grew day by day | `get-post-timeline` with `post_id` |
| Day by day totals | `get-daily-metrics` |
| When to post | `get-best-time-to-post` |
| How fast posts fade | `get-content-decay` |
| Does posting more help | `get-posting-frequency` |
| Follower counts and growth | `get-follower-stats` (`granularity`: `daily`, `weekly`, `monthly`) |
| One account's stored history | `get-account-analytics` |
| Audience age, gender, location | `get-account-demographics`, or the network tools below |
| A network-specific report | `get-account-report` with a report name, run live on the network |
| Feed changes into a warehouse | `get-analytics-delta` with a cursor (cursors expire after 7 days) |

`list-post-analytics` defaults to the last 90 days and covers at most 366. `get-daily-metrics` defaults to 180 days. `get-best-time-to-post` and `get-content-decay` default to the last 365 days. `tags` on `list-post-analytics` filters Socialit posts by tag.

## Network insights

These keep each network's own vocabulary and often come live from the network:

- **YouTube**: `get-youtube-channel-insights`, `get-youtube-daily-views`, `get-youtube-video-retention`, `get-youtube-demographics`.
- **Instagram**: `get-instagram-account-insights`, `get-instagram-follower-history`, `get-instagram-demographics`, `get-instagram-online-followers`, and `get-instagram-story-insights` for live Stories.
- **Facebook**: `get-facebook-page-insights`, `get-facebook-demographics`, `get-facebook-post-reactions`, `get-facebook-post-earnings`.
- **LinkedIn**: `get-linkedin-org-analytics` (Company Pages), `get-linkedin-aggregate-analytics` (personal profiles), `get-linkedin-post-analytics`, `get-linkedin-post-reactions`.
- **TikTok**: `get-tiktok-account-insights`.
- **Google Business Profile**: `get-google-business-performance`, `get-google-business-search-keywords`.

Limits, delays, and parameters for each are in `references/network-insights.md`. Deeper network analytics (YouTube Analytics reporting, Reddit traffic, Pinterest audience insights) are Focus mode resources: `get-focus-manifest` lists what an account offers.

## Inbox analytics

`get-inbox-volume`, `get-inbox-heatmap`, `get-inbox-source-breakdown`, `get-inbox-response-time`, `get-inbox-top-accounts`, `list-conversation-analytics`, and `get-conversation-analytics` measure the inbox. They require `from_date`. `source` separates human replies from automated ones (`human`, `api`, `workflow`, `sequence`, `broadcast`, `comment_automation`, and more). `get-inbox-response-time` returns the median, p90, and p99 time to first response in seconds; convert to minutes or hours. Load `socialit-inbox` for the conversations themselves.

## Freshness and syncing

Socialit syncs analytics in the background. When numbers look stale:

- `get-post-analytics` answers `202` with the same body while a sync is pending, and queues one. Check `sync_status` (`synced`, `pending`, `partial`, `unavailable`) and `last_synced_at`, then try again shortly.
- `sync-account-analytics` queues a refresh of one account (`scope`: `stats`, `insights`, `posts`). It returns `202`, and `analytics.synced` fires when it finishes. The account must be active.
- `sync-external-posts` fetches an account's latest posts from the network, including ones posted natively, and imports their metrics. With `url` or `post_id` it returns that post. It calls the network, so do not loop it.

YouTube Analytics and Google Business Profile data arrive 2 to 3 days late. Say so when the last days look empty.

## Network rules that change the math

- **YouTube** figures are never merged into cross-network totals or rates. Aggregates carry `calculated: true` and a `platform_breakdown`. Report YouTube on its own line.
- **Pinterest** analytics are fetched live and never stored, so stored aggregates (dashboard, daily metrics, best time) exclude Pinterest. Use `get-account-analytics` for a Pinterest account.
- **Facebook post earnings** are lifetime, cumulative values in the network's raw unit. Never sum them across posts or dates.
- **Facebook Page demographics** have country and city only; Meta removed age and gender for Pages.
- **Instagram demographics** need at least 100 followers and return the top 45 entries per dimension.

## Presenting results

- Lead with the answer, then the evidence. "Your best post in October was the launch video on LinkedIn: 4.1% engagement rate, 2.3x your average."
- Always state the range, the accounts or profile, and the time zone.
- Use a table for comparisons across posts, accounts, or networks. Round rates to one decimal and large counts to thousands (12.4K).
- Show the sample size. A best-time slot based on 2 posts is a hint, not a finding.
- Compare like with like: a rate with a rate, the same period length, the same networks. With `compare: "previous_period"`, show the change in percent.
- Name gaps instead of hiding them: a `null` metric means the network does not report it.
- Do not claim cause and effect from correlations. Posting frequency and best time describe the past.

### Best time to post

`get-best-time-to-post` returns `slots` with `day_of_week` (0 = Sunday), `hour`, `avg_engagement` (average engagements per post, not a rate), and `post_count`, all in **UTC**, best first. Convert each slot to the user's time zone before showing it (the day can change), skip slots with very few posts, and offer the top 3 to 5. To act on them, the `socialit-publishing` skill can set queue slots at those times after the user agrees.

`get-inbox-heatmap` numbers days differently: `dow` 1 = Monday to 7 = Sunday, hours in UTC.

### A monthly report

1. `get-analytics-dashboard` for the month with `compare: "previous_period"`.
2. `get-follower-stats` with `granularity: "weekly"`.
3. `list-post-analytics` sorted by `engagement_rate`, top 5 and bottom 3.
4. `get-best-time-to-post` and `get-posting-frequency` for recommendations.
5. Network sections where they matter (YouTube on its own, Pinterest from `get-account-analytics`).

Write it as: summary (3 bullets), totals table with change, top posts table, followers, what to try next.
