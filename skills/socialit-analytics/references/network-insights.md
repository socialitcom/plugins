# Network insights

Each tool takes `account_id` (an `sa_` account on that network) unless noted. Ranges use `from_date` and `to_date` (UTC days). Most default to the last 30 days, and your plan's history limit applies on top of the maximum below. Missing network permissions answer `403 missing_scopes` with a `reconnect_url`: send the user through it (it keeps the account and its history).

## YouTube

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-youtube-channel-insights` | Channel metrics from YouTube Analytics (`metrics`, `metric_type`) | Max 89 days, data is 2 to 3 days late. Needs the YouTube Analytics permission |
| `get-youtube-daily-views` | Daily views, watch time, and subscriber changes of one video (`video_id`) | Max 90 days, by default ending 3 days ago |
| `get-youtube-video-retention` | The audience retention curve of one video (`video_id`) | |
| `get-youtube-demographics` | Age and gender as viewer percentages, country as views (`breakdown`: `age`, `gender`, `country`) | Channel-wide, or one video with `video_id` |

YouTube engagement rate is always `null`, and YouTube figures stay out of cross-network totals.

## Instagram

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-instagram-account-insights` | Account metrics (`metrics`, `metric_type`, `breakdown`) | Max 89 days. Only `reach` supports `metric_type: "time_series"`; breakdowns (`media_product_type`, `follow_type`, `follower_type`, `contact_button_type`) need `total_value` |
| `get-instagram-follower-history` | Daily follower count | Max 89 days |
| `get-instagram-demographics` | Audience by `age`, `city`, `country`, `gender` | Needs at least 100 followers; top 45 entries per dimension. `metric`: `follower_demographics` or `engaged_audience_demographics`; `timeframe`: `this_week` or `this_month` |
| `get-instagram-online-followers` | Followers online per hour | |
| `list-instagram-stories`, `get-instagram-story-insights` | Active Stories and their insights | Only active Stories are listed, so read them while they are live |

## Facebook

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-facebook-page-insights` | Page metrics; `followers_gained` and `followers_lost` are built from daily follows and unfollows | Max 89 days |
| `get-facebook-demographics` | Followers by `country` and `city` | No age or gender (Meta removed them for Pages) |
| `get-facebook-post-reactions` | Reaction breakdown of one post (`post_id`) | |
| `get-facebook-post-earnings` | Lifetime monetization earnings of one post (`post_id`) | Cumulative, raw unit. Never sum across posts or dates |

## LinkedIn

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-linkedin-org-analytics` | Share, follower, and page statistics of a Company Page | Max 89 days |
| `get-linkedin-aggregate-analytics` | Totals (`aggregation: "TOTAL"`) or a daily series (`"DAILY"`) across a personal profile's posts. `metrics`: `IMPRESSION`, `MEMBERS_REACHED`, `REACTION`, `COMMENT`, `RESHARE`, `POST_SAVE`, `POST_SEND` | Only posts published through Socialit (a LinkedIn API limit) |
| `get-linkedin-post-analytics` | Stats of one post (`urn`) | |
| `get-linkedin-post-reactions` | Individual reactions on a Company Page post (`urn`, `limit`, `cursor`) | |

## TikTok

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-tiktok-account-insights` | Live counters plus daily history from Socialit snapshots | Max 89 days |

## Google Business Profile

| Tool | Returns | Range and notes |
| --- | --- | --- |
| `get-google-business-performance` | Daily impressions, clicks, calls, directions, conversations, bookings, and food orders (`metrics`) | Up to 18 months back, data is 2 to 3 days late |
| `get-google-business-search-keywords` | Search keywords people used to find the location (`start_month`, `end_month` as `YYYY-MM`) | Monthly |

## Any network

| Tool | Returns |
| --- | --- |
| `get-account-analytics` | Stored daily follower snapshots and normalized daily insights for any account (Pinterest is read live and cached for an hour) |
| `get-account-demographics` | Live audience demographics (age, gender, country, city, locale where the network has them) |
| `get-account-report` | A live, network-specific report by name; extra arguments pass through to the report |
| `get-follower-stats` | Follower counts and growth across accounts (`account_ids` comma separated, `profile_id`, `platform`, `granularity`) |

## Aggregates in detail

| Tool | Shape |
| --- | --- |
| `get-analytics-dashboard` | `totals`, `previous_totals`, `followers`, a `daily` series, `platform_breakdown`, `top_posts`, `recent_posts`, `data_as_of`. Daily and total metrics count when the engagement happened |
| `get-daily-metrics` | Per day `post_count` and summed metrics. `attribution: "publish"` (default) credits a post's metrics to the day it was published; `"received"` credits each day with the growth that happened that day |
| `get-best-time-to-post` | `slots` of `day_of_week` (0 = Sunday), `hour`, `avg_engagement`, `post_count`, in UTC |
| `get-content-decay` | Average share of a post's final engagement reached after 1h, 6h, 24h, 3d, 7d, and 30d |
| `get-posting-frequency` | Average engagement rate and engagement per platform for each posts-per-week cadence, with `weeks_count` |
| `list-post-analytics` | Paginated posts with metrics per platform plus an `overview` |
