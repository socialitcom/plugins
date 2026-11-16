# Inbox support by network

Support depends on what each network's API allows. When a network does not offer an action, the tool returns 422 `not_supported`: tell the user and do not retry.

## Direct messages

| Capability | Networks |
| --- | --- |
| Conversations in the inbox | Facebook (Messenger), Instagram, X, Bluesky, Reddit, Discord, Mastodon, TikTok business accounts |
| Start a conversation (`create-conversation`) | X, Bluesky, Reddit, Discord, Mastodon. Not possible on Facebook, Instagram, Threads, LinkedIn, YouTube, Pinterest, or Google Business Profile: the person must write first |
| Message requests (`folder: "requests"`) | Facebook, Instagram |
| Accept a request (`accept-conversation`) | Facebook, Instagram, Bluesky, TikTok |
| Read receipts from `mark-conversation-read` | Messenger, Instagram, Bluesky, TikTok, Reddit, Mastodon |
| Reactions (`add-message-reaction`, `remove-message-reaction`) | Facebook, Instagram, Bluesky, Discord |
| Delete or unsend (`delete-message`) | X, Reddit, Bluesky (removes it for the account only) |
| Thread control (`change-thread-control`) | Facebook (Meta handover protocol) |

### Messaging windows (Facebook and Instagram)

Meta limits when a business may message a person. Socialit measures from the person's last message (`last_inbound_at` on the conversation, `last_inbound_at` and `messaging_window_open` on a contact channel).

| Time since the person's last message | What is allowed |
| --- | --- |
| Up to 24 hours | Any message |
| 24 hours to 7 days | Only `message_tag: "HUMAN_AGENT"`, for replies written or approved by a person on the team. Instagram accepts no other tag, and Meta retired the other Messenger tags, so Socialit refuses them |
| More than 7 days | Nothing until the person writes again |

Automated sends (broadcasts, sequences, workflows, comment automations) check the window before calling Meta and record the recipient as skipped with `outside_messaging_window` instead of sending.

## Comments

| Capability | Networks |
| --- | --- |
| Read and reply | Most publishing networks |
| Hide and unhide | Facebook, Instagram, Threads, X (replies to the account's posts), YouTube (hiding rejects the comment), TikTok, Bluesky (replies in threads the account started) |
| Moderation status (`set-comment-moderation`) | YouTube |
| Pin (`pin-comment`) | TikTok, Reddit |
| Private reply by DM (`send-private-reply`) | Facebook, Instagram: one per comment, within 7 days |
| Edit text (`update-comment` with `text`) | Reddit, and the account's own comments on Facebook, YouTube, LinkedIn |
| Like a post (`like-post`) | LinkedIn (with `reaction_type`: `LIKE`, `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION`, `ENTERTAINMENT`), X, Facebook, YouTube, Bluesky, Instagram. Not Reddit |

`list-inbox-posts` with `platform: "metaads"` lists only boosted and dark posts from Meta ads.

## Reviews and mentions

| Capability | Networks |
| --- | --- |
| Reviews (`list-reviews`, `reply-to-review`, `delete-review-reply`) | Google Business Profile, Facebook recommendations |
| Mentions (`list-mentions`) | Where the network reports them |
| Reply to a mention (`reply-to-mention`) | Instagram, Facebook, Threads, LinkedIn, TikTok, and networks that answer a mention as a comment reply |

Mention `kind` values: `post`, `comment`, `caption`, `story`, `reply`, `quote`, `tag`.

## Real time or polled

Socialit receives the networks' own webhooks (Meta, X, LinkedIn, TikTok, Google Business, Discord, YouTube) and stores new items in near real time. Reddit, Bluesky, Mastodon, Threads, and LinkedIn organizations are polled every 5 to 30 minutes, as are some kinds on other networks (for example Facebook reviews and TikTok mentions).

`sync-inbox` queues an immediate poll. Pass `account_id` for one account and `kinds` (`conversations`, `comments`, `reviews`, `mentions`) to limit it. It returns right away (`queued`); new items appear shortly after.

## Errors you may see

| Error | Meaning |
| --- | --- |
| 422 `not_supported` | The network does not offer this action |
| 409 `account_needs_reconnect` | The network revoked the account's access; reconnect it (see `socialit-get-started`) |
| 402 `enterprise_required` | The X endpoint needs X Enterprise access |
| 422 validation error on `snoozed_until` | Snoozing needs a time in the future |
