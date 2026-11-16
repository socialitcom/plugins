# Network rules for publishing

Socialit validates every post against these limits before it schedules it. They are the practical limits Socialit enforces, which are sometimes lower than a network's absolute maximum. `get-post-platform-options` returns the full `platform_options` schema of every network; this page lists the options people ask about most.

## At a glance

| Network | Key | Text limit | Media per post | Media required | First comment | Edit after publishing | Delete on network |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Facebook | `facebook` | 63,206 | Up to 10 photos or 1 video, not mixed | No | Feed and Reels | Yes | Yes |
| Instagram | `instagram` | 2,200 (30 hashtags, 20 mentions) | 1 image or video, or a 2 to 10 item carousel (mixed) | Yes | Not on Stories | No (caption is fixed) | Yes |
| X | `x` (alias `twitter`) | 280 weighted, 25,000 with Premium | 4 images, or 1 GIF, or 1 video | No | Yes | Premium only, within 30 minutes, up to 5 edits, single posts | Yes |
| LinkedIn | `linkedin` | 3,000 | Up to 20 images, or 1 video, or 1 document (PDF, PPT, PPTX, DOC, DOCX) | No | Yes | Text, call to action, and landing page | Yes |
| Threads | `threads` | 500 (up to 5 links) | 1 image or video, or a 2 to 20 item carousel (mixed) | No | Yes | No | Yes |
| Bluesky | `bluesky` | 300 graphemes | 4 images or 1 video | No | Yes | No | Yes |
| Pinterest | `pinterest` | 800 (title 100) | 1 image, a 2 to 5 image carousel, or 1 video | Yes | No | Yes | Yes |
| TikTok | `tiktok` | 2,200 (photo title 90, photo description 4,000) | 1 video or 1 to 35 photos | Yes | No | No | No |
| YouTube | `youtube` | Description 5,000 bytes, title 100 | Exactly 1 video | Yes | Yes | Yes, or `update-post-metadata` | Yes |
| Google Business Profile | `google_business` (alias `googlebusiness`) | 1,500 | 1 photo or 1 video | No | No | Yes | Yes |
| Reddit | `reddit` | Title 300, body 40,000 | 1 image, a 2 to 20 image gallery, or 1 video | No | Yes | Body of text posts only (titles are fixed) | Yes |
| Snapchat | `snapchat` | 160 (Spotlight description) | 1 image or video (Saved Stories take several) | Yes | No | No | Saved Stories and Spotlight only |
| Discord | `discord` | 2,000 | Up to 10 attachments of any type | No | Yes | Yes (not polls) | Yes |
| Mastodon | `mastodon` | 500 by default, set per server | 4 images, or 1 video or audio file | No | Yes | Yes (visibility is fixed) | Yes |

Networks without edit or delete refuse the call with `edit_not_supported`, `unpublish_not_supported`, or `capability_not_supported`. To change a post there, unpublish it (where possible) and publish a new one, after the user agrees.

## Choices the user must make

Some options have no default on purpose. Ask the user; never pick for them.

- **TikTok** `privacyLevel`: one of the options returned by `get-tiktok-creator-info` (`PUBLIC_TO_EVERYONE`, `MUTUAL_FOLLOW_FRIENDS`, `FOLLOWER_OF_CREATOR`, `SELF_ONLY`). `allowComment`, `allowDuet`, and `allowStitch` default to off. `contentPreviewConfirmed: false` or `expressConsentGiven: false` blocks publishing; set them only after the user has seen the preview and agreed. `get-tiktok-composer-requirements` returns what TikTok requires a posting screen to show (privacy options, interaction toggles, disclosure, and the declaration text); relay it to the user.
- **YouTube** `selfDeclaredMadeForKids`: the "made for kids" answer. Also confirm `privacyStatus` (`public`, `unlisted`, `private`), which defaults to `public`.

## Options by network

### Facebook

- `contentType`: `feed` (text, link, photos, video), `reel` (one vertical 3 to 90 second video), or `story` (one photo or video up to 60 seconds, no caption, 24 hours).
- `link` for a link preview, `title` for video and Reel titles, `thumbnailUrl` for a custom cover.
- `draft: true` creates an unpublished draft in Meta's own tools instead of publishing.

### Instagram

- `contentType`: `auto` (1 image is feed, 1 video is a Reel, 2 to 10 items a carousel), `feed`, `reel`, `story`, or `carousel`. Every single video is a Reel; `shareToFeed` also shows it in the feed.
- `collaborators` (up to 3 usernames), `userTags`, `locationId`, `coverUrl` for a Reel cover, `altText`.
- Instagram allows 100 published posts per account per 24 hours. `get-instagram-publishing-limit` shows what is left.

### X

- `replySettings`: `following`, `mentionedUsers`, `subscribers`, or `verified` (omit for everyone).
- `poll` (cannot combine with media, quotes, or threads), `communityId`, `autoThread` to split long text into a thread, `premium: true` for 25,000 characters on Premium accounts.
- Publishing to X is paid from prepaid X credits: `402 insufficient_credits` means they ran out.

### LinkedIn

- `organizationUrn` to post as a Company Page (`list-linkedin-organizations`), `visibility` (`PUBLIC`, `CONNECTIONS`, `LOGGED_IN`).
- `documentTitle` for document posts, `disableLinkPreview`, `poll`, `reshareUrl` to repost.

### Threads

- `replyControl`: `everyone`, `accounts_you_follow`, `mentioned_only`, `parent_post_author_only`, or `followers_only`.
- `topicTag` (one per post), `poll` (2 to 4 options), `linkAttachment` for a link card on text posts.
- Threads allows 250 posts per 24 hours, and the first comment counts toward it.

### Bluesky

- `langs` (up to 3), `labels` for content warnings, `threadgate` for reply controls, `linkCard: false` to drop the automatic card.

### Pinterest

- `boardId`: the destination board. It defaults to the account's default board, but send it on every post. `list-pinterest-boards` lists boards; `set-default-pinterest-board` changes the default.
- `title`, `link` (click-through URL), `altText`.

### TikTok

- See the required choices above. `draft: true` sends the video to the creator's TikTok inbox to finish in the app instead of posting.
- `title` and `description` for photo posts, `videoCoverTimestampMs` for the cover frame.
- Disclosures: `commercialContentType` (`brand_organic` for the user's own brand, `brand_content` for a paid partnership). Branded content cannot be `SELF_ONLY`.
- Video: 3 to 600 seconds (the creator's own maximum may be lower, see `get-tiktok-creator-info`).

### YouTube

- `title` (defaults to the first line of `content`), `description`, `tags`, `categoryId`, `privacyStatus`, `selfDeclaredMadeForKids`.
- `publishAt` hands scheduling to YouTube (the video uploads as private). Socialit's own `scheduled_at` does not need it.
- `playlistId` (`list-youtube-playlists`), `thumbnailUrl` (the channel must be verified), `containsSyntheticMedia` for AI content.
- Vertical or square videos up to 180 seconds publish as Shorts. YouTube API calls cost quota units, so extras such as captions and playlists use more of the daily quota.

### Google Business Profile

- `topicType`: `STANDARD`, `EVENT` (needs `event`), or `OFFER` (needs `event` for the dates, usually `offer`).
- `callToAction` for a button. Replying to reviews is covered by the `socialit-inbox` skill.

### Reddit

- `subreddit` (without `r/`, defaults to the account's default subreddit) and `title` (defaults to the first line of `content`, and cannot be edited later).
- `flairId` or `flairText`, `nsfw`, `spoiler`, `kind` to force a post type.
- Check first: `validate-subreddit`, `get-reddit-subreddit-rules`, and `list-reddit-flairs`. Many communities require flair or ban links.

### Snapchat

- `contentType`: `story`, `saved_story`, or `spotlight` (video only, needs `locale` such as `en_US`).
- Video 5 to 60 seconds, at least 540 by 960 pixels, 9:16 recommended. Saved Story titles up to 45 characters.

### Discord

- `channelId` (defaults to the connected channel; `list-discord-channels`), `forumThreadName` (required for forum channels), `embeds`, `poll`.
- `mentions` defaults to `none`, so `@everyone` in the text pings nobody unless set. Confirm before pinging a server.

### Mastodon

- `visibility` (`public`, `unlisted`, `private`, `direct`), `spoiler_text` for a content warning (counts toward the limit), `language`, `poll`.
- Character and media limits come from the user's server.

## Media checks

- `validate-media-url` with a `url` or `media_id` and `platforms` returns size and format results per network.
- Networks that need media (Instagram, TikTok, Pinterest, YouTube, Snapchat) reject text-only posts.
- Never mix images and video where the table says "not mixed"; split into separate posts or use `targets` overrides.
