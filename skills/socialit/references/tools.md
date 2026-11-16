# Socialit MCP tools

Every tool of the hosted server at `https://api.socialit.com/mcp`, grouped by toolset: 674 tools in 25 toolsets. This file is generated from the server catalog; do not edit it by hand.

Permissions: `read` tools change nothing, `write` tools create and change content (posts, media, inbox replies, contacts, automations), and `admin` tools manage the workspace (profiles, API keys, webhooks, request logs, connect links, publishing settings). An API key or OAuth connection only lists the tools its permissions allow, and the signed-in user's role applies on top for OAuth.

Kinds: read (no changes), create, update, action (changes state), delete, destructive (cannot be undone), external (acts on a social network or calls a URL), send (reaches real people on a network). Clients use them to ask before calling a tool.

Trim the list with `?toolsets=posts,media` on the server URL or the `X-MCP-Toolsets` header, using the toolset names in the headings below. The `workspace` toolset is always included. `?read_only=true` or `X-MCP-Readonly: true` keeps only read tools.

On OAuth connections every tool outside the `workspace` toolset also takes an optional `workspace` argument (a workspace ID or exact name from `list-workspaces`) that runs that one call in another allowed workspace. List tools take `page`, `limit` and `filters: [{ field, condition, value }]`.

Focus mode also has typed toolsets per network (`focus_youtube`, `focus_reddit`, and so on, about 2,000 tools). They are not listed here or by default; select them by name, for example `?toolsets=focus,focus_youtube`.

## Workspace (`workspace`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `get-current-workspace` | read | read | The workspace this connection acts on by default, your role and the granted permissions. Every connection has it. |
| `list-workspaces` | read | read | Workspaces this OAuth connection may use, with your role in each. OAuth only. |
| `switch-workspace` | update | read | Change the default workspace for later calls on this OAuth connection. OAuth only. |

## Invitations, Users (`team`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `create-invite-token` | create | admin | Create an invite link. |
| `list-users` | read | read | List users. |
| `get-user` | read | read | Get a user. |

## API Keys (`api_keys`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `verify-credential` | read | read | Verify the current credential. |
| `list-api-keys` | read | admin | List API keys. |
| `create-api-key` | create | admin | Create an API key. |
| `get-api-key` | read | admin | Get an API key. |
| `update-api-key` | update | admin | Update an API key. |
| `delete-api-key` | delete | admin | Delete an API key. |
| `regenerate-api-key` | destructive | admin | Regenerate an API key. |

## Webhooks, Events (`webhooks`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-webhooks` | read | admin | List webhooks. |
| `create-webhook` | create | admin | Create a webhook. |
| `get-webhook` | read | admin | Get a webhook. |
| `update-webhook` | update | admin | Update a webhook. |
| `delete-webhook` | delete | admin | Delete a webhook. |
| `test-webhook` | external | admin | Send a test event. |
| `reset-webhook-secret` | destructive | admin | Rotate the signing secret. |
| `list-all-webhook-deliveries` | read | admin | List webhook deliveries. |
| `get-webhook-delivery` | read | admin | Get a webhook delivery. |
| `redeliver-webhook-delivery` | external | admin | Redeliver an event. |
| `list-webhook-deliveries` | read | admin | List a webhook's deliveries. |
| `retry-failed-webhook-deliveries` | external | admin | Retry failed deliveries. |
| `list-event-types` | read | read | List event types. |
| `list-events` | read | read | List events. |
| `get-event` | read | read | Get an event. |

## Logs (`logs`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-request-logs` | read | admin | List API request logs. |
| `get-request-log` | read | admin | Get an API request log. |
| `list-activity` | read | admin | List workspace activity. |

## Accounts (`accounts`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-profiles` | read | read | List profiles. |
| `create-profile` | create | admin | Create a profile. |
| `get-profile` | read | read | Get a profile. |
| `update-profile` | update | admin | Update a profile. |
| `delete-profile` | destructive | admin | Delete a profile. |
| `list-account-groups` | read | read | List account groups. |
| `create-account-group` | create | write | Create an account group. |
| `get-account-group` | read | read | Get an account group. |
| `update-account-group` | update | write | Update an account group. |
| `delete-account-group` | delete | write | Delete an account group. |
| `add-account-group-accounts` | update | write | Add accounts to an account group. |
| `remove-account-group-account` | update | write | Remove an account from an account group. |
| `list-accounts` | read | read | List connected accounts. |
| `get-accounts-health` | read | read | Health of every account. |
| `get-follower-stats` | read | read | Follower counts and growth. |
| `get-account` | read | read | Get an account. |
| `update-account` | update | write | Update an account or move it to another profile. |
| `disconnect-account` | destructive | write | Disconnect an account. |
| `get-account-health` | external | read | Check the health of an account. |
| `list-account-platform-posts` | external | read | List posts published on the network. |
| `list-connect-platforms` | read | read | List connectable networks. |
| `get-connect-url` | external | write | Start connecting an account (authorization URL). |
| `complete-connect` | create | write | Complete a connect flow with code and state. |
| `connect-with-credentials` | create | write | Connect an account with credentials. |
| `get-pending-connection` | read | write | Get a pending connection. |
| `select-pending-connection` | create | write | Connect the selected targets of a pending connection. |
| `list-facebook-page-connect-targets` | read | write | List FacebookPage targets of a pending connection. |
| `select-facebook-page-connect-targets` | create | write | Connect the selected FacebookPage targets. |
| `list-instagram-account-connect-targets` | read | write | List InstagramAccount targets of a pending connection. |
| `select-instagram-account-connect-targets` | create | write | Connect the selected InstagramAccount targets. |
| `list-linkedin-organization-connect-targets` | read | write | List LinkedinOrganization targets of a pending connection. |
| `select-linkedin-organization-connect-targets` | create | write | Connect the selected LinkedinOrganization targets. |
| `list-pinterest-board-connect-targets` | read | write | List PinterestBoard targets of a pending connection. |
| `select-pinterest-board-connect-targets` | create | write | Connect the selected PinterestBoard targets. |
| `list-google-business-location-connect-targets` | read | write | List GoogleBusinessLocation targets of a pending connection. |
| `select-google-business-location-connect-targets` | create | write | Connect the selected GoogleBusinessLocation targets. |
| `list-snapchat-profile-connect-targets` | read | write | List SnapchatProfile targets of a pending connection. |
| `select-snapchat-profile-connect-targets` | create | write | Connect the selected SnapchatProfile targets. |
| `connect-bluesky-credentials` | create | write | Connect Bluesky with an app password. |
| `connect-discord-channels` | create | write | Connect Discord channels. |
| `list-connect-links` | read | admin | List hosted connect links. |
| `create-connect-link` | create | admin | Create a hosted connect link. |
| `get-connect-link` | read | admin | Get a hosted connect link. |
| `revoke-connect-link` | delete | admin | Revoke a hosted connect link. |

## Posts (`posts`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-posts` | read | read | List posts. |
| `create-post` | create | write | Create a post. |
| `get-post-platform-options` | read | read | Publish option schemas per network. |
| `bulk-upload-posts` | create | write | Bulk create posts from CSV. |
| `get-post` | read | read | Get a post. |
| `update-post` | update | write | Update a post. |
| `delete-post` | delete | write | Delete a post. |
| `schedule-post` | action | write | Schedule, queue or publish a draft. |
| `cancel-post` | destructive | write | Cancel a scheduled post. |
| `retry-post` | action | write | Retry failed postings. |
| `unpublish-post` | external | write | Delete a published post on the networks. |
| `edit-published-post` | external | write | Edit a published post on the network. |
| `update-post-metadata` | external | write | Update video metadata (YouTube). |
| `duplicate-post` | create | write | Duplicate a post. |
| `approve-post` | action | write | Approve a pending post. |
| `reject-post` | action | write | Reject a pending post. |
| `list-post-notes` | read | read | List internal notes of a post. |
| `create-post-note` | create | write | Add an internal note to a post. |
| `update-post-note` | update | write | Edit a post note. |
| `delete-post-note` | delete | write | Delete a post note. |
| `list-postings` | read | read | List postings. |
| `get-posting` | read | read | Get a posting. |
| `get-publishing-settings` | read | read | Get publishing settings. |
| `update-publishing-settings` | update | admin | Update publishing settings. |

## Queues (`queues`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-queues` | read | read | List queues. |
| `create-queue` | create | write | Create a queue. |
| `preview-queue` | read | read | Preview upcoming free slots. |
| `get-next-queue-slot` | read | read | Get the next free slot. |
| `get-queue` | read | read | Get a queue. |
| `update-queue` | update | write | Update a queue. |
| `delete-queue` | delete | write | Delete a queue. |

## Media (`media`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `presign-media-upload` | create | write | Get an upload URL. |
| `create-media-from-url` | create | write | Register media from a public URL. |
| `list-media-library` | read | read | List the media library. |
| `get-media-usage` | read | read | Media storage usage. |
| `list-media-folders` | read | read | List media folders. |
| `get-media-item` | read | read | Get media. |
| `update-media-item` | update | write | Update media. |
| `complete-media-upload` | action | write | Finalize a presigned upload. |
| `delete-media-item` | delete | write | Delete media. |

## Templates (`templates`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-templates` | read | read | List templates. |
| `create-template` | create | write | Create a template. |
| `get-template` | read | read | Get a template. |
| `update-template` | update | write | Update a template. |
| `delete-template` | delete | write | Delete a template. |

## Tools (`tools`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `validate-post-length` | read | read | Character counts per network. |
| `validate-post` | read | read | Validate a post for its target networks. |
| `validate-media-url` | read | read | Validate a media URL or library item. |
| `validate-subreddit` | read | read | Check a subreddit. |
| `download-tik-tok-video` | read | read | TikTok video metadata. |
| `submit-feedback` | create | read | Send feedback to the Socialit team. |
| `list-changelog` | read | read | API changelog. |

## Inbox (`inbox`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-conversations` | read | read | List conversations. |
| `create-conversation` | create | write | Start a conversation. |
| `search-conversations` | read | read | Search conversations. |
| `get-conversation` | read | read | Get a conversation. |
| `update-conversation` | update | write | Update a conversation. |
| `accept-conversation` | send | write | Accept a message request. |
| `mark-conversation-read` | action | write | Mark a conversation as read. |
| `mark-conversation-unread` | action | write | Mark a conversation as unread. |
| `send-typing-indicator` | action | write | Send a typing indicator. |
| `change-thread-control` | action | write | Change who answers a conversation. |
| `list-messages` | read | read | List messages. |
| `send-message` | send | write | Send a message. |
| `delete-message` | delete | write | Delete a message. |
| `add-message-reaction` | action | write | React to a message. |
| `remove-message-reaction` | delete | write | Remove a reaction. |
| `get-message-attachment` | read | read | Resolve a message attachment. |
| `list-inbox-posts` | read | read | List posts with comments. |
| `get-inbox-post` | read | read | Get a commented post. |
| `list-post-comments` | read | read | List a post's comments. |
| `create-comment` | send | write | Comment on a post or reply to a comment. |
| `like-post` | action | write | Like a post. |
| `unlike-post` | delete | write | Unlike a post. |
| `list-comments` | read | read | List comments. |
| `get-comment` | read | read | Get a comment. |
| `update-comment` | update | write | Edit or triage a comment. |
| `delete-comment` | delete | write | Delete a comment. |
| `reply-to-comment` | send | write | Reply to a comment. |
| `set-comment-moderation` | action | write | Set comment moderation status. |
| `hide-comment` | action | write | Hide a comment. |
| `unhide-comment` | delete | write | Unhide a comment. |
| `pin-comment` | action | write | Pin a comment. |
| `unpin-comment` | delete | write | Unpin a comment. |
| `like-comment` | action | write | Like a comment. |
| `unlike-comment` | delete | write | Unlike a comment. |
| `send-private-reply` | send | write | Send a private reply. |
| `list-reviews` | read | read | List reviews. |
| `get-review` | read | read | Get a review. |
| `update-review` | update | write | Triage a review. |
| `reply-to-review` | send | write | Reply to a review. |
| `delete-review-reply` | delete | write | Delete a review reply. |
| `list-mentions` | read | read | List mentions. |
| `get-mention` | read | read | Get a mention. |
| `update-mention` | update | write | Triage a mention. |
| `reply-to-mention` | send | write | Reply to a mention. |
| `get-inbox-counts` | read | read | Inbox counters. |
| `sync-inbox` | action | write | Sync the inbox now. |
| `get-messenger-menu` | read | read | Get persistent menu. |
| `set-messenger-menu` | update | write | Set persistent menu. |
| `delete-messenger-menu` | delete | write | Delete persistent menu. |
| `get-messenger-get-started` | read | read | Get Messenger Get Started button. |
| `set-messenger-get-started` | update | write | Set Messenger Get Started button. |
| `delete-messenger-get-started` | delete | write | Delete Messenger Get Started button. |
| `get-messenger-greeting` | read | read | Get Messenger greeting text. |
| `set-messenger-greeting` | update | write | Set Messenger greeting text. |
| `delete-messenger-greeting` | delete | write | Delete Messenger greeting text. |
| `get-messenger-ice-breakers` | read | read | Get Messenger ice breakers. |
| `set-messenger-ice-breakers` | update | write | Set Messenger ice breakers. |
| `delete-messenger-ice-breakers` | delete | write | Delete Messenger ice breakers. |
| `get-instagram-ice-breakers` | read | read | Get Instagram ice breakers. |
| `set-instagram-ice-breakers` | update | write | Set Instagram ice breakers. |
| `delete-instagram-ice-breakers` | delete | write | Delete Instagram ice breakers. |
| `retweet` | external | write | Repost (retweet) a post on X. |
| `unretweet` | delete | write | Undo a repost on X. |
| `bookmark-tweet` | external | write | Bookmark a post on X. |
| `remove-bookmark` | delete | write | Remove an X bookmark. |
| `follow-x-user` | external | write | Follow a user on X. |
| `unfollow-x-user` | delete | write | Unfollow a user on X. |
| `search-tweets` | read | read | Search recent posts on X. |
| `get-tweet` | read | read | Look up a post on X. |
| `search-reddit` | read | read | Search Reddit posts. |
| `get-reddit-feed` | read | read | Get a subreddit feed. |
| `get-reddit-post-comments` | read | read | Get a Reddit post's comments. |

## Analytics (`analytics`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-post-analytics` | read | read | List post analytics. |
| `get-post-analytics` | read | read | Get analytics for one post. |
| `get-analytics-delta` | read | read | Analytics changed since a cursor. |
| `get-post-timeline` | read | read | Get the daily metric history of a post. |
| `sync-external-posts` | external | write | Sync external posts of an account now. |
| `get-analytics-dashboard` | read | read | Get an analytics dashboard. |
| `get-daily-metrics` | read | read | Get daily aggregated metrics. |
| `get-best-time-to-post` | read | read | Get the best times to post. |
| `get-content-decay` | read | read | Get content performance decay. |
| `get-posting-frequency` | read | read | Get posting frequency vs engagement. |
| `get-account-analytics` | read | read | Get stored analytics of an account. |
| `get-account-demographics` | read | read | Get audience demographics of an account. |
| `get-account-report` | read | read | Run a network analytics report. |
| `sync-account-analytics` | action | write | Queue an analytics sync for an account. |
| `get-youtube-channel-insights` | read | read | Get YouTube channel insights. |
| `get-youtube-daily-views` | read | read | Get YouTube daily views of a video. |
| `get-youtube-video-retention` | read | read | Get the YouTube audience retention curve of a video. |
| `get-youtube-demographics` | read | read | Get YouTube audience demographics. |
| `get-linkedin-org-analytics` | read | read | Get LinkedIn organization analytics. |
| `get-linkedin-aggregate-analytics` | read | read | Get LinkedIn personal account aggregate stats. |
| `get-linkedin-post-analytics` | read | read | Get LinkedIn post stats. |
| `get-linkedin-post-reactions` | read | read | Get LinkedIn post reactions. |
| `get-tiktok-account-insights` | read | read | Get TikTok account insights. |
| `get-facebook-page-insights` | read | read | Get Facebook Page insights. |
| `get-facebook-post-earnings` | read | read | Get lifetime monetization earnings of a Facebook post. |
| `get-facebook-demographics` | read | read | Get Facebook Page follower demographics. |
| `get-facebook-post-reactions` | read | read | Get the reaction breakdown of a Facebook post. |
| `get-instagram-account-insights` | read | read | Get Instagram account insights. |
| `get-instagram-follower-history` | read | read | Get Instagram follower history. |
| `get-instagram-demographics` | read | read | Get Instagram audience demographics. |
| `get-instagram-online-followers` | read | read | Get Instagram followers online per hour. |
| `get-google-business-performance` | read | read | Get Google Business Profile performance metrics. |
| `get-google-business-search-keywords` | read | read | Get Google Business Profile search keywords. |
| `get-inbox-volume` | read | read | Get inbox messaging volume. |
| `get-inbox-heatmap` | read | read | Get the inbox day × hour heatmap. |
| `get-inbox-source-breakdown` | read | read | Get inbox messages by source. |
| `get-inbox-response-time` | read | read | Get inbox response-time stats. |
| `get-inbox-top-accounts` | read | read | Get top accounts by inbox volume. |
| `list-conversation-analytics` | read | read | List conversation analytics. |
| `get-conversation-analytics` | read | read | Get analytics of one conversation. |

## Contacts (`contacts`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-contacts` | read | read | List contacts. |
| `create-contact` | create | write | Create a contact. |
| `bulk-create-contacts` | create | write | Bulk create contacts. |
| `bulk-tag-contacts` | action | write | Bulk tag contacts. |
| `bulk-delete-contacts` | destructive | write | Bulk delete contacts. |
| `list-contact-tags` | read | read | List contact tags. |
| `get-contact` | read | read | Get a contact. |
| `update-contact` | update | write | Update a contact. |
| `delete-contact` | delete | write | Delete a contact. |
| `update-contact-tags` | update | write | Add or remove contact tags. |
| `list-contact-channels` | read | read | List a contact's channels. |
| `create-contact-channel` | create | write | Link a channel to a contact. |
| `update-contact-channel` | update | write | Update a contact channel. |
| `delete-contact-channel` | delete | write | Unlink a contact channel. |
| `set-contact-field-value` | update | write | Set a custom field value. |
| `clear-contact-field-value` | update | write | Clear a custom field value. |

## Custom Fields (`custom_fields`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-custom-fields` | read | read | List custom fields. |
| `create-custom-field` | create | write | Create a custom field. |
| `get-custom-field` | read | read | Get a custom field. |
| `update-custom-field` | update | write | Update a custom field. |
| `delete-custom-field` | delete | write | Delete a custom field. |

## Broadcasts (`broadcasts`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-broadcasts` | read | read | List broadcasts. |
| `create-broadcast` | create | write | Create a broadcast draft. |
| `preview-broadcast-audience` | read | read | Preview a broadcast audience. |
| `get-broadcast` | read | read | Get a broadcast. |
| `update-broadcast` | update | write | Update a broadcast. |
| `delete-broadcast` | delete | write | Delete a broadcast. |
| `send-broadcast` | send | write | Send a broadcast now. |
| `schedule-broadcast` | action | write | Schedule a broadcast. |
| `cancel-broadcast` | destructive | write | Cancel a broadcast. |
| `list-broadcast-recipients` | read | read | List broadcast recipients. |
| `add-broadcast-recipients` | action | write | Add broadcast recipients. |

## Sequences (`sequences`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-sequences` | read | read | List sequences. |
| `create-sequence` | create | write | Create a sequence. |
| `get-sequence` | read | read | Get a sequence. |
| `update-sequence` | update | write | Update a sequence. |
| `delete-sequence` | delete | write | Delete a sequence. |
| `activate-sequence` | action | write | Activate a sequence. |
| `pause-sequence` | action | write | Pause a sequence. |
| `enroll-sequence-contacts` | action | write | Enroll contacts in a sequence. |
| `unenroll-sequence-contact` | delete | write | Unenroll a contact. |
| `list-sequence-enrollments` | read | read | List sequence enrollments. |

## Workflows (`workflows`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-workflows` | read | read | List workflows. |
| `create-workflow` | create | write | Create a workflow. |
| `list-workflow-node-types` | read | read | List workflow node types. |
| `get-workflow` | read | read | Get a workflow. |
| `update-workflow` | update | write | Update a workflow. |
| `delete-workflow` | delete | write | Delete a workflow. |
| `activate-workflow` | action | write | Activate a workflow. |
| `pause-workflow` | action | write | Pause a workflow. |
| `duplicate-workflow` | create | write | Duplicate a workflow. |
| `list-workflow-versions` | read | read | List workflow versions. |
| `get-workflow-version` | read | read | Get a workflow version. |
| `restore-workflow-version` | action | write | Restore a workflow version. |
| `list-workflow-executions` | read | read | List workflow runs. |
| `start-workflow-execution` | action | write | Start a workflow run manually. |
| `trigger-workflow` | action | write | Trigger an API workflow. |
| `get-workflow-execution` | read | read | Get a workflow run. |
| `cancel-workflow-execution` | destructive | write | Stop a workflow run. |
| `list-workflow-execution-events` | read | read | Get a run timeline. |

## Comment Automations (`comment_automations`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-comment-automations` | read | read | List comment automations. |
| `create-comment-automation` | create | write | Create a comment automation. |
| `get-comment-automation` | read | read | Get a comment automation. |
| `update-comment-automation` | update | write | Update a comment automation. |
| `delete-comment-automation` | delete | write | Delete a comment automation. |
| `list-comment-automation-logs` | read | read | List comment automation logs. |

## Ads: accounts & measurement (`ads_measurement`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `update-ad-connection` | update | write | Update an ads connection. |
| `discover-ad-connection-accounts` | action | write | Re-discover ad accounts. |
| `list-ad-businesses` | read | read | List business portfolios. |
| `list-ad-business-users` | read | read | List business users. |
| `list-ad-business-centers` | read | read | List TikTok Business Centers. |
| `list-ad-page-users` | read | read | List Page users. |
| `assign-ad-page-user` | create | admin | Assign a user to a Page. |
| `remove-ad-page-user` | delete | admin | Remove a user from a Page. |
| `get-ad-manager-hierarchy` | read | read | Get the Google Ads manager hierarchy. |
| `create-ad-manager-link` | create | admin | Invite a client account to a manager. |
| `update-ad-manager-link` | update | admin | Accept, decline, cancel or end a manager link. |
| `list-ad-accounts` | read | read | List ad accounts. |
| `create-ad-account` | create | admin | Create a Meta ad account. |
| `get-ad-account` | read | read | Get an ad account. |
| `update-ad-account` | update | write | Select or move an ad account. |
| `update-ad-account-settings` | update | write | Update ad account settings on the network. |
| `get-ad-account-finance` | read | read | Get ad account finances. |
| `get-ad-account-live` | read | read | Read campaigns and ad sets live. |
| `get-ad-account-dsa-defaults` | read | read | Get DSA defaults. |
| `get-ad-account-dsa-recommendations` | read | read | Get DSA recommendations. |
| `list-ad-account-users` | read | read | List ad account users. |
| `assign-ad-account-user` | create | admin | Assign a user to an ad account. |
| `remove-ad-account-user` | delete | admin | Remove a user from an ad account. |
| `list-ad-account-activity` | read | read | Ad account change log. |
| `list-ad-studies` | read | read | List A/B tests and lift studies. |
| `list-ad-instagram-accounts` | read | read | List Instagram ad identities. |
| `list-ad-instagram-posts` | read | read | List Instagram posts to boost. |
| `list-advertisable-applications` | read | read | List advertisable apps. |
| `get-ios-campaign-limits` | read | read | Get iOS 14 campaign limits. |
| `list-ad-pixels` | read | read | List TikTok pixels. |
| `list-custom-conversions` | read | read | List custom conversions. |
| `create-custom-conversion` | create | write | Create a custom conversion. |
| `list-ad-labels` | read | read | List ad labels. |
| `create-ad-label` | create | write | Create an ad label. |
| `update-ad-label` | update | write | Update an ad label. |
| `delete-ad-label` | delete | write | Remove an ad label. |
| `attach-ad-label` | action | write | Attach a label. |
| `detach-ad-label` | delete | write | Detach a label. |
| `list-high-demand-periods` | read | read | List high-demand periods. |
| `create-high-demand-period` | create | write | Schedule a budget increase. |
| `list-value-rule-sets` | read | read | List value rule sets. |
| `create-value-rule-set` | create | write | Create a value rule set. |
| `get-value-rule-set` | read | read | Read a value rule set. |
| `replace-value-rule-set` | update | write | Replace a value rule set. |
| `delete-value-rule-set` | delete | write | Delete a value rule set. |
| `list-negative-keyword-lists` | read | read | List negative keyword lists. |
| `create-negative-keyword-list` | create | write | Create a negative keyword list. |
| `get-negative-keyword-list` | read | read | Get a negative keyword list. |
| `rename-negative-keyword-list` | update | write | Rename a negative keyword list. |
| `delete-negative-keyword-list` | delete | write | Delete a negative keyword list. |
| `replace-negative-keyword-list-keywords` | update | write | Replace negative list keywords. |
| `list-account-callouts` | read | read | List account callouts. |
| `add-account-callouts` | create | write | Add account callouts. |
| `update-account-callouts` | update | write | Update account callouts. |
| `remove-account-callout` | delete | write | Remove an account callout. |
| `list-account-sitelinks` | read | read | List account sitelinks. |
| `add-account-sitelinks` | create | write | Add account sitelinks. |
| `update-account-sitelinks` | update | write | Update account sitelinks. |
| `remove-account-sitelink` | delete | write | Remove an account sitelink. |
| `list-account-structured-snippets` | read | read | List account structured snippets. |
| `add-account-structured-snippets` | create | write | Add account structured snippets. |
| `update-account-structured-snippets` | update | write | Update account structured snippets. |
| `remove-account-structured-snippet` | delete | write | Remove an account structured snippet. |
| `list-ad-comments` | read | read | List comments on an ad. |
| `reply-ad-comment` | create | write | Reply to an ad comment. |
| `hide-ad-comment` | action | write | Hide or unhide an ad comment. |
| `delete-ad-comment` | delete | write | Delete an ad comment. |
| `get-ad-tracking-params` | read | read | Get ad URL tracking params. |
| `set-ad-tracking-params` | update | write | Set ad URL tracking params. |
| `list-tracking-tags` | read | read | List tracking tags. |
| `create-tracking-tag` | create | write | Create a tracking tag. |
| `get-tracking-tag` | read | read | Get a tracking tag. |
| `update-tracking-tag` | update | write | Update a tracking tag. |
| `list-tracking-tag-shared-accounts` | read | read | List accounts a tag is shared with. |
| `share-tracking-tag` | create | write | Share a tag with an ad account. |
| `unshare-tracking-tag` | delete | write | Stop sharing a tag with an account. |
| `list-tracking-tag-events` | read | read | List conversion events of a tag. |
| `create-tracking-tag-event` | create | write | Create a conversion event. |
| `update-tracking-tag-event` | update | write | Update a conversion event. |
| `delete-tracking-tag-event` | delete | write | Delete a conversion event. |
| `list-tracking-tag-users` | read | read | List tag users. |
| `assign-tracking-tag-user` | create | admin | Assign a user to a tag. |
| `remove-tracking-tag-user` | delete | admin | Remove a user from a tag. |
| `list-tracking-tag-partners` | read | read | List partner businesses of a tag. |
| `get-tracking-tag-diagnostics` | read | read | Get tag diagnostics. |
| `get-tracking-tag-stats` | read | read | Get aggregated tag stats. |
| `list-leads` | read | read | List leads. |
| `get-lead` | read | read | Get a lead. |
| `list-lead-forms` | read | read | List lead forms. |
| `create-lead-form` | create | write | Create a lead form. |
| `get-lead-form` | read | read | Get a lead form. |
| `archive-lead-form` | delete | write | Archive a lead form. |
| `list-lead-form-leads` | read | read | List the leads of a form. |
| `create-test-lead` | create | write | Create a test lead. |
| `delete-test-leads` | delete | write | Delete test leads. |
| `list-catalogs` | read | read | List product catalogs. |
| `create-catalog` | create | write | Create a product catalog. |
| `get-catalog` | read | read | Get a product catalog. |
| `delete-catalog` | delete | write | Delete a product catalog. |
| `list-catalog-products` | read | read | List catalog products. |
| `create-catalog-product` | create | write | Add a product. |
| `batch-catalog-products` | action | write | Create, update or delete products in bulk. |
| `get-catalog-batch` | read | read | Get a bulk request’s status. |
| `get-catalog-product` | read | read | Get a product. |
| `update-catalog-product` | update | write | Update a product. |
| `delete-catalog-product` | delete | write | Delete a product. |
| `list-catalog-feeds` | read | read | List product feeds. |
| `create-catalog-feed` | create | write | Create a product feed. |
| `list-catalog-feed-uploads` | read | read | List a feed’s uploads. |
| `create-catalog-feed-upload` | action | write | Fetch a feed file now. |
| `list-product-sets` | read | read | List product sets. |
| `create-product-set` | create | write | Create a product set. |
| `update-product-set` | update | write | Update a product set. |
| `delete-product-set` | delete | write | Delete a product set. |
| `send-conversions` | send | write | Send conversion events. |
| `adjust-conversions` | send | write | Adjust uploaded conversions. |
| `get-event-match-quality` | read | read | Get Event Match Quality. |
| `list-conversion-actions` | read | read | List conversion actions. |
| `create-conversion-action` | create | write | Create a website conversion action. |
| `update-conversion-action` | update | write | Update a conversion action. |
| `list-conversion-goals` | read | read | List account conversion goals. |
| `update-conversion-goals` | update | write | Update account conversion goals. |
| `list-custom-conversion-goals` | read | read | List custom conversion goals. |
| `create-custom-conversion-goal` | create | write | Create a custom conversion goal. |
| `update-custom-conversion-goal` | update | write | Update a custom conversion goal. |
| `delete-custom-conversion-goal` | delete | write | Remove a custom conversion goal. |
| `list-conversion-destinations` | read | read | List conversion destinations. |
| `create-conversion-destination` | create | write | Create a conversion destination. |
| `get-conversion-destination` | read | read | Get a conversion destination. |
| `update-conversion-destination` | update | write | Update a conversion destination. |
| `delete-conversion-destination` | delete | write | Delete (disable) a conversion destination. |
| `list-conversion-associations` | read | read | List associated campaigns. |
| `add-conversion-associations` | action | write | Associate campaigns. |
| `remove-conversion-associations` | delete | write | Remove associated campaigns. |
| `get-conversion-metrics` | read | read | Get attribution metrics. |

## Ads (`ads`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `connect-ads` | external | write | Connect ads for a platform. |
| `set-tiktok-ads-brand-identity` | update | write | Set TikTok brand identity. |
| `connect-openai-ads` | create | write | Connect an OpenAI (ChatGPT) Ads account. |
| `connect-whop-ads` | create | write | Connect a Whop account. |
| `list-ad-connections` | read | read | List ads connections. |
| `get-ad-connection` | read | read | Get an ads connection. |
| `delete-ad-connection` | delete | write | Disconnect an ads connection. |
| `sync-ad-connection` | action | write | Queue an ads sync. |
| `list-ads` | read | read | List ads. |
| `list-ad-campaigns` | read | read | List campaigns. |
| `list-ad-sets` | read | read | List ad sets. |
| `get-ad-tree` | read | read | Get the campaign tree. |
| `get-ad-timeline` | read | read | Get daily account metrics. |
| `create-ad-campaign` | create | write | Create a campaign. |
| `get-ad-campaign` | read | read | Get campaign details. |
| `update-ad-campaign` | update | write | Update a campaign. |
| `delete-ad-campaign` | delete | write | Delete a campaign. |
| `set-ad-campaign-status` | update | write | Pause or resume a campaign. |
| `bulk-set-ad-campaign-status` | action | write | Pause or resume many campaigns. |
| `duplicate-ad-campaign` | create | write | Duplicate a campaign. |
| `get-ad-campaign-analytics` | read | read | Get campaign analytics. |
| `get-ad-campaign-bidding` | read | read | Read a campaign's bidding. |
| `get-ad-campaign-schedule` | read | read | Read a campaign's ad schedule. |
| `replace-ad-campaign-schedule` | update | write | Replace a campaign's ad schedule. |
| `get-ad-campaign-targeting` | read | read | Read a Google campaign's targeting. |
| `replace-ad-campaign-targeting` | update | write | Edit a Google campaign's targeting. |
| `list-ad-campaign-negative-keywords` | read | read | List campaign negative keywords. |
| `replace-ad-campaign-negative-keywords` | update | write | Replace campaign negative keywords. |
| `list-ad-campaign-negative-keyword-lists` | read | read | List campaign negative keyword lists. |
| `replace-ad-campaign-negative-keyword-lists` | update | write | Replace campaign negative keyword lists. |
| `list-ad-campaign-assets` | read | read | List campaign assets. |
| `add-ad-campaign-assets` | create | write | Attach campaign assets. |
| `update-ad-campaign-assets` | update | write | Update campaign assets. |
| `remove-ad-campaign-assets` | delete | write | Remove campaign assets. |
| `get-ad-campaign-conversion-goals` | read | read | Get campaign conversion goals. |
| `update-ad-campaign-conversion-goals` | update | write | Update campaign conversion goals. |
| `list-ad-asset-groups` | read | read | List Performance Max asset groups. |
| `create-ad-asset-group` | create | write | Create a Performance Max asset group. |
| `get-ad-asset-group` | read | read | Get a Performance Max asset group. |
| `update-ad-asset-group` | update | write | Update a Performance Max asset group. |
| `delete-ad-asset-group` | delete | write | Remove a Performance Max asset group. |
| `link-ad-asset-group-assets` | action | write | Link or unlink asset group assets. |
| `replace-ad-listing-group-filters` | update | write | Replace an asset group's listing-group tree. |
| `create-ad-set` | create | write | Create an ad set. |
| `get-ad-set` | read | read | Get ad set details. |
| `update-ad-set` | update | write | Update an ad set. |
| `delete-ad-set` | delete | write | Delete an ad set. |
| `set-ad-set-status` | update | write | Pause or resume an ad set. |
| `duplicate-ad-set` | create | write | Duplicate an ad set. |
| `list-ad-set-assets` | read | read | List ad-group assets. |
| `add-ad-set-assets` | create | write | Attach ad-group assets. |
| `update-ad-set-assets` | update | write | Update ad-group assets. |
| `remove-ad-set-assets` | delete | write | Remove ad-group assets. |
| `create-ad` | create | write | Create an ad (full tree). |
| `boost-ad-post` | create | write | Boost a post as an ad. |
| `create-messaging-ad` | create | write | Create a click-to-message ad. |
| `create-call-ad` | create | write | Create a click-to-call ad. |
| `get-ad` | read | read | Get an ad. |
| `update-ad` | update | write | Update an ad. |
| `delete-ad` | delete | write | Delete an ad. |
| `set-ad-status` | update | write | Pause or resume an ad. |
| `duplicate-ad` | create | write | Duplicate an ad. |
| `get-ad-analytics` | read | read | Get ad analytics. |
| `get-ad-previews` | read | read | Render previews of an ad. |
| `get-ad-review` | read | read | Read an ad's review verdict. |
| `get-ad-media` | read | read | Direct URLs of an ad's media. |
| `preview-ad-creative` | read | read | Render pre-create ad previews. |
| `list-ad-creatives` | read | read | Creative library. |
| `create-ad-creative` | create | write | Create a standalone creative. |
| `get-ad-creative` | read | read | Creative details. |
| `rename-ad-creative` | update | write | Rename a creative. |
| `delete-ad-creative` | delete | write | Delete a creative. |
| `upload-ad-image` | create | write | Upload an ad image. |
| `list-ad-images` | read | read | Ad image library. |
| `upload-ad-video` | create | write | Upload an ad video. |
| `list-ad-videos` | read | read | Ad video library. |
| `get-ad-video` | read | read | Get ad video processing status. |
| `delete-ad-video` | delete | write | Delete an ad video. |
| `list-tiktok-ad-identities` | read | read | List TikTok ad identities. |
| `list-ad-partnership-content` | read | read | List partnership ad content. |
| `list-ad-partnership-permissions` | read | read | List partnership permissions. |
| `set-ad-partnership-permission` | action | write | Set a partnership permission. |
| `get-ad-insights` | read | read | Flexible live insights query. |
| `create-ad-insights-report` | create | write | Submit an async insights report. |
| `get-ad-insights-report` | read | read | Poll an async insights report. |
| `get-tiktok-smart-plus-materials` | read | read | TikTok Smart+ per-creative performance. |
| `get-ad-search-terms` | read | read | Google Ads search terms report. |
| `list-local-services-leads` | read | read | Google Local Services Ads leads. |
| `list-local-services-lead-conversations` | read | read | List Local Services lead conversations. |
| `generate-ad-keyword-ideas` | read | read | Generate keyword ideas. |
| `get-ad-keyword-historical-metrics` | read | read | Get historical keyword metrics. |
| `list-ad-keywords` | read | read | List Search keywords. |
| `add-ad-keywords` | create | write | Add Search keywords. |
| `update-ad-keyword` | update | write | Pause or enable a Search keyword. |
| `delete-ad-keyword` | delete | write | Remove a Search keyword. |
| `list-ad-recommendations` | read | read | List Google Ads recommendations. |
| `apply-ad-recommendations` | action | write | Apply Google Ads recommendations. |
| `dismiss-ad-recommendations` | action | write | Dismiss Google Ads recommendations. |
| `list-ad-bid-strategies` | read | read | List portfolio bid strategies. |
| `create-ad-bid-strategy` | create | write | Create a portfolio bid strategy. |
| `update-ad-bid-strategy` | update | write | Update a portfolio bid strategy. |
| `list-ad-shared-budgets` | read | read | List shared budgets. |
| `create-ad-shared-budget` | create | write | Create a shared budget. |
| `search-ad-interests` | read | read | Search targeting interests. |
| `search-ad-targeting` | read | read | Search targeting options. |
| `browse-ad-targeting` | read | read | Browse targeting categories. |
| `estimate-ad-reach` | read | read | Estimate audience reach. |
| `get-ad-bid-pricing` | read | read | Suggested bid and budget bounds. |
| `get-ad-supply-forecast` | read | read | Forecast ad delivery. |
| `list-ad-audiences` | read | read | List custom audiences. |
| `create-ad-audience` | create | write | Create a custom audience. |
| `get-ad-audience` | read | read | Get audience details. |
| `update-ad-audience` | update | write | Update an audience. |
| `delete-ad-audience` | delete | write | Delete a custom audience. |
| `add-ad-audience-users` | action | write | Add users to an audience. |
| `replace-ad-audience-companies` | action | write | Replace audience companies. |
| `search-ad-library` | read | read | Search the public Ad Library. |
| `create-ad-reach-frequency-prediction` | create | write | Create a reach & frequency prediction. |
| `get-ad-reach-frequency-prediction` | read | read | Get a reach & frequency prediction. |
| `cancel-ad-reach-frequency-prediction` | delete | write | Cancel a reach & frequency booking. |
| `reserve-ad-reach-frequency-prediction` | action | write | Reserve reach & frequency inventory. |

## Platform tools (`platform_tools`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-gmb-reviews` | read | read | List Google Business reviews. |
| `batch-get-gmb-reviews` | read | read | Batch get Google Business reviews. |
| `get-gmb-review` | read | read | Get a Google Business review. |
| `reply-to-gmb-review` | send | write | Reply to a Google Business review. |
| `delete-gmb-review-reply` | delete | write | Delete a Google Business review reply. |
| `get-gmb-automation` | read | read | Get the Google Business automation opt-in. |
| `update-gmb-automation` | update | write | Update the Google Business automation opt-in. |
| `list-gmb-automation-audit` | read | read | List the Google Business automation audit log. |
| `get-gmb-location-details` | read | read | Get Google Business location details. |
| `update-gmb-location-details` | update | write | Update Google Business location details. |
| `get-gmb-attributes` | read | read | Get Google Business attributes. |
| `update-gmb-attributes` | update | write | Update Google Business attributes. |
| `get-gmb-attribute-metadata` | read | read | Get Google Business attribute metadata. |
| `get-gmb-services` | read | read | Get Google Business services. |
| `replace-gmb-services` | update | write | Replace Google Business services. |
| `get-gmb-food-menus` | read | read | Get Google Business food menus. |
| `update-gmb-food-menus` | update | write | Update Google Business food menus. |
| `list-gmb-media` | read | read | List Google Business photos and videos. |
| `upload-gmb-media` | create | write | Upload a Google Business photo or video. |
| `delete-gmb-media` | delete | write | Delete a Google Business photo or video. |
| `list-gmb-place-actions` | read | read | List Google Business action links. |
| `create-gmb-place-action` | create | write | Create a Google Business action link. |
| `update-gmb-place-action` | update | write | Update a Google Business action link. |
| `delete-gmb-place-action` | delete | write | Delete a Google Business action link. |
| `get-gmb-verifications` | read | read | Get Google Business verification state. |
| `fetch-gmb-verification-options` | read | write | Fetch Google Business verification options. |
| `start-gmb-verification` | external | write | Start a Google Business verification. |
| `complete-gmb-verification` | external | write | Complete a Google Business verification. |
| `list-gmb-locations` | read | read | List Google Business locations of the login. |
| `select-gmb-location` | update | write | Switch the Google Business location. |
| `assign-gmb-location` | action | write | Assign a Google Business location to a profile. |
| `list-facebook-pages` | read | read | List Facebook Pages of the connection. |
| `select-facebook-page` | update | write | Switch the Facebook Page. |
| `get-facebook-webhook-subscription` | read | read | Read the Facebook Page's webhook subscription. |
| `resubscribe-facebook-webhooks` | action | write | Re-subscribe the Facebook Page to Socialit webhooks. |
| `list-facebook-business-partners` | read | read | List partner businesses of the Page. |
| `share-facebook-page-with-partner` | action | write | Share the Page with a partner business. |
| `revoke-facebook-business-partner` | delete | write | Revoke a partner business from the Page. |
| `list-linkedin-organizations` | read | read | List LinkedIn organizations. |
| `switch-linkedin-account-type` | update | write | Switch LinkedIn account type. |
| `resolve-linkedin-mention` | read | read | Resolve a LinkedIn mention. |
| `list-instagram-stories` | read | read | List active Instagram stories. |
| `get-instagram-story-insights` | read | read | Get Instagram story insights. |
| `get-instagram-publishing-limit` | read | read | Get Instagram publishing limit. |
| `search-instagram-audio` | read | read | Search Instagram audio. |
| `get-instagram-audio` | read | read | Get Instagram audio metadata. |
| `instagram-business-discovery` | read | read | Look up a public Instagram Business account. |
| `get-instagram-follow-status` | read | read | Check whether an Instagram user follows the account. |
| `list-pinterest-boards` | read | read | List Pinterest boards. |
| `create-pinterest-board` | create | write | Create a Pinterest board. |
| `set-default-pinterest-board` | update | write | Set the default Pinterest board. |
| `list-youtube-playlists` | read | read | List YouTube playlists. |
| `create-youtube-playlist` | create | write | Create a YouTube playlist. |
| `set-default-youtube-playlist` | update | write | Set the default YouTube playlist. |
| `get-youtube-transcript` | read | read | Get a YouTube video transcript. |
| `list-reddit-subreddits` | read | read | List Reddit subreddits. |
| `set-default-reddit-subreddit` | update | write | Set the default subreddit. |
| `get-reddit-subreddit-rules` | read | read | Get subreddit rules. |
| `list-reddit-flairs` | read | read | List subreddit post flairs. |
| `set-reddit-post-flair` | action | write | Set a Reddit post flair. |
| `vote-on-reddit` | action | write | Vote on a Reddit post or comment. |
| `get-tiktok-creator-info` | read | read | Get TikTok creator info. |
| `get-tiktok-composer-requirements` | read | read | Get TikTok composer requirements. |
| `list-tiktok-commercial-music` | read | read | List trending TikTok commercial music. |
| `search-tiktok-locations` | read | read | Search TikTok location tags. |
| `get-bluesky-settings` | read | read | Get Bluesky account settings. |
| `update-bluesky-settings` | update | write | Update Bluesky account settings. |
| `get-discord-settings` | read | read | Get Discord account settings. |
| `update-discord-settings` | update | write | Update Discord account settings. |
| `list-discord-channels` | read | read | List Discord server channels. |
| `send-discord-dm` | send | write | Send a Discord direct message. |
| `list-discord-roles` | read | read | List Discord server roles. |
| `create-discord-role` | create | write | Create a Discord server role. |
| `update-discord-role` | update | write | Edit a Discord server role. |
| `delete-discord-role` | delete | write | Delete a Discord server role. |
| `list-discord-members` | read | read | List Discord server members. |
| `search-discord-members` | read | read | Search Discord server members. |
| `get-discord-member` | read | read | Get a Discord server member. |
| `add-discord-member-role` | action | write | Assign a role to a Discord member. |
| `remove-discord-member-role` | delete | write | Remove a role from a Discord member. |
| `delete-discord-message` | delete | write | Delete a Discord channel message. |
| `crosspost-discord-message` | external | write | Crosspost a Discord announcement. |
| `create-discord-thread` | create | write | Create a Discord public thread. |
| `list-discord-pins` | read | read | List pinned Discord messages. |
| `pin-discord-message` | action | write | Pin a Discord message. |
| `unpin-discord-message` | delete | write | Unpin a Discord message. |
| `list-discord-events` | read | read | List Discord scheduled events. |
| `create-discord-event` | create | write | Create a Discord scheduled event. |
| `get-discord-event` | read | read | Get a Discord scheduled event. |
| `update-discord-event` | update | write | Update a Discord scheduled event. |
| `delete-discord-event` | delete | write | Delete a Discord scheduled event. |

## Focus (`focus`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-focus-platforms` | read | read | List networks with Focus mode. |
| `get-focus-manifest` | read | read | Get the Focus manifest of an account. |
| `list-focus-items` | read | read | List focus items (or get a singleton resource). |
| `create-focus-item` | create | write | Create a focus item. |
| `update-focus-singleton` | update | write | Update a singleton focus resource. |
| `delete-focus-singleton` | delete | write | Delete or reset a singleton focus resource. |
| `get-focus-item` | read | read | Get a focus item. |
| `update-focus-item` | update | write | Update a focus item. |
| `delete-focus-item` | delete | write | Delete a focus item. |
| `run-focus-action` | action | write | Run a focus action. |

## Billing (`billing`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `get-usage` | read | read | Get usage. Workspace role: owner, admin, billing. |
| `get-x-api-usage` | read | read | Get X API usage. Workspace role: owner, admin, billing. |
| `get-billing` | read | read | Get billing overview. Workspace role: owner, admin, billing. |
| `list-plans` | read | read | List plans. Workspace role: owner, admin, billing. |
| `get-x-pricing` | read | read | Get X API pricing. Workspace role: owner, admin, billing. |
| `get-billing-credits` | read | read | Get prepaid credits. Workspace role: owner, admin, billing. |
| `preview-billing-change` | read | read | Preview a plan change. Workspace role: owner, admin, billing. |
| `list-billing-invoices` | read | read | List invoices. Workspace role: owner, admin, billing. |
| `get-billing-invoice` | read | read | Get an invoice. Workspace role: owner, admin, billing. |

## Support Assistant (`support`)

| Tool | Kind | Permission | Description |
| --- | --- | --- | --- |
| `list-support-runs` | read | read | List a thread's support answers. |
| `ask-support-question` | external | write | Ask a support question. |
| `get-support-run` | read | read | Get a support answer. |
