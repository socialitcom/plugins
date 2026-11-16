# Socialit webhook event types

Types are `<object>.<past_tense_verb>` and are never renamed; new ones are added as features land. `list-event-types` returns the live catalog with each type's group and description. Subscribe with `events: [...]` on `create-webhook` or `update-webhook`, or with `all_events: true`.

Every delivery has the same envelope: `id` (`evt_...`), `object` (`event`), `type`, `created_at`, `workspace_id`, `test`, and `data`. The tables below show what is inside `data`. Most payloads embed the full object in its API shape; a few carry only IDs, written as `{ id }`.

## Posts

| Event | When | `data` |
| --- | --- | --- |
| `post.scheduled` | A post was scheduled | `{ post }` |
| `post.publishing` | A post started publishing to its accounts | `{ post }` |
| `post.published` | Published to every target account | `{ post }` |
| `post.partially_published` | Published to some accounts, failed on others | `{ post }` |
| `post.failed` | Failed on every target account | `{ post }` |
| `post.deleted` | Deleted from Socialit | `{ post }` |
| `post.cancelled` | A scheduled or publishing post was cancelled, or every published target was unpublished | `{ post }` |
| `post.unpublished` | A published post was removed from the network | `{ post, posting }` |
| `post.edited` | A published post was edited on the network | `{ post, posting }` |
| `post.recycled` | An evergreen post was cloned and rescheduled | `{ post, source_post_id }` (`post` is the clone) |
| `post.approval_requested` | A post is waiting for approval | `{ post, approval }` |
| `post.approved` | A pending post was approved | `{ post, approval, comment }` |
| `post.rejected` | A pending post was rejected and moved back to draft | `{ post, approval, comment }` |
| `posting.published` | One target account finished publishing, before the post roll-up | `{ post, posting }` |
| `posting.failed` | One target account failed permanently (retryable failures stay quiet) | `{ post, posting }` |

## Accounts

| Event | When | `data` |
| --- | --- | --- |
| `account.connected` | A social account was connected or reconnected | `{ account, reconnected, connect_flow }` |
| `account.disconnected` | A social account was disconnected | `{ account, disconnection_type, reason }` (`intentional` or `unintentional`) |
| `account.needs_reconnect` | The account lost its authorization and must be reconnected | `{ account, reason, platform_code }` |
| `account.updated` | Label, defaults, or profile assignment changed | `{ account, changes }` |

## Inbox

| Event | When | `data` |
| --- | --- | --- |
| `message.received` | A direct message arrived | `{ message, conversation, account }` |
| `message.sent` | A message was sent, from Socialit or echoed from the network's own app | `{ message, conversation, account }` |
| `message.delivered`, `message.read` | A sent message was delivered or read | `{ message, conversation, account }` |
| `message.failed` | A sent message failed to deliver | `{ message, conversation, account, error }` |
| `message.edited` | A message was edited (with its edit history) | `{ message, conversation, account }` |
| `message.deleted` | A message was deleted or unsent | `{ message, conversation, account }` |
| `reaction.received` | A participant added or removed a reaction | `{ reaction, message, conversation, account }`; `reaction` has `emoji`, `action` (`added` or `removed`), `message_id`, `sender`, `reacted_at` |
| `conversation.created` | A new conversation started | `{ conversation, account }` |
| `conversation.updated` | Status, assignment, or read state changed | `{ conversation, account, changes }` |
| `conversation.control_changed` | The app answering a conversation changed (Meta handover) | `{ conversation, account, control }`; `control` has `owner`, `previous_owner`, `owner_app_id` |
| `comment.received` | A comment arrived on a post | `{ comment, post, account }` |
| `comment.replied` | A reply was posted to a comment | `{ comment, post, account }` |
| `comment.hidden`, `comment.unhidden` | A comment was hidden or made visible again | `{ comment, post, account }` |
| `comment.deleted` | A comment was deleted | `{ comment, post, account }` |
| `review.received` | A review arrived | `{ review, account }` |
| `review.updated` | A reviewer edited the text or rating | `{ review, account }` |
| `review.replied` | A reply was posted to a review | `{ review, account }` |
| `mention.received` | The account was mentioned | `{ mention, account }` |

The first sync of a newly connected account imports its history silently: no inbox events fire for old items.

## Contacts

| Event | When | `data` |
| --- | --- | --- |
| `contact.created` | A contact was created | `{ contact, source }` |
| `contact.updated` | A contact was updated | `{ contact, changes, source }` |
| `contact.deleted` | A contact was deleted | `{ contact: { id, name } }` |
| `contact.tag_added`, `contact.tag_removed` | One tag was added or removed (fires once per tag) | `{ contact: { id }, tag, source }` |
| `contact.field_changed` | A custom field value changed (not on create) | `{ contact: { id }, field, previous_value, value, source }` |

## Messaging (broadcasts, sequences, workflows)

| Event | When | `data` |
| --- | --- | --- |
| `broadcast.scheduled` | A broadcast was scheduled | `{ broadcast }` |
| `broadcast.sending` | A broadcast started sending | `{ broadcast }` |
| `broadcast.sent` | A broadcast finished sending | `{ broadcast }` |
| `broadcast.failed` | A broadcast failed | `{ broadcast, error }` |
| `broadcast.cancelled` | A scheduled or sending broadcast was cancelled | `{ broadcast }` |
| `sequence.enrolled` | A contact was enrolled | `{ sequence: { id, name }, contact: { id }, enrollment: { id } }` |
| `sequence.step_sent` | A step was sent to a contact | same, plus `step: { order }` and `message_id` |
| `sequence.completed` | A contact finished every step | same as `sequence.enrolled` |
| `sequence.exited` | A contact left the sequence, finished or not | same, plus `exit_reason`: `completed`, `replied`, `manual`, `failed`, `unsubscribed`, `sequence_deleted` |
| `workflow.triggered` | A workflow run started | `{ workflow: { id, name }, execution: { id, status }, conversation, contact, trigger: { type, text } }` |
| `workflow.completed` | A run ended; `execution.status` is `completed`, or `exited` for a run ended on purpose | same as `workflow.triggered` |
| `workflow.failed` | A run failed | same, plus `error: { node_id, message }` |

## Ads

| Event | When | `data` |
| --- | --- | --- |
| `ad.created` | An ad was created through Socialit | `{ ad, campaign, ad_set }` |
| `ad.updated` | An ad was updated through Socialit | `{ ad, changes }` |
| `ad.deleted` | An ad was deleted or archived through Socialit | `{ ad }` |
| `ad.status_changed` | A campaign, ad set, or ad changed effective status | `{ level, object, previous_status, status, platform_status, error, connection, ad_account }` |
| `ad.video.processed` | An ad video finished processing (ready or error) | `{ video, connection, ad_account }` |
| `ad_campaign.created`, `ad_campaign.deleted` | A campaign was created, or deleted or archived | `{ campaign }` |
| `ad_campaign.updated` | Name, budget, bidding, or schedule changed | `{ campaign, changes }` |
| `ad_audience.created`, `ad_audience.deleted` | A custom audience was created or deleted | `{ audience }` |
| `ad_insights_report.completed` | An async insights report finished (completed or failed) | `{ report }` |
| `ad_connection.connected` | An ads connection was created or reconnected | `{ connection, ad_accounts, reconnected }` |
| `ad_connection.disconnected` | An ads connection was disconnected | `{ connection, reason }` |
| `ad_connection.needs_reconnect` | An ads connection lost its authorization | `{ connection, reason, platform_code }` |
| `ad_account.initial_sync_completed` | First sync and 90 day backfill finished | `{ connection, ad_accounts, sync }` |
| `ad_account.sync_failed` | An ad account has not synced for 24 hours | `{ connection, ad_account, sync }` |
| `ad_account.sync_recovered` | A failing ad account synced again | `{ connection, ad_account, sync }` |
| `ad_account.created` | An ad account was created or found again | `{ ad_account, connection }` |
| `ad_account.updated` | Selection, profile, settings, name, status, or currency changed | `{ ad_account, changes }` |
| `lead.received` | A lead form submission was stored (test leads included) | `{ lead, form: { id, name }, ad_account }` |

## Analytics, support assistant, workspace, billing

| Event | When | `data` |
| --- | --- | --- |
| `analytics.synced` | An account's analytics sync finished (roughly hourly). It carries no metrics; pull them with the analytics tools | `{ account, sync: { synced_at, posts_updated } }` |
| `support_run.completed` | The support assistant finished; `run.status` is `completed`, or `needs_human` when handed to a person | `{ run }` |
| `support_run.failed` | A support question failed or expired | `{ run }` |
| `member.joined`, `member.removed` | A user joined or was removed from the workspace | `{ member: { id, user_id, email, role } }` |
| `webhook.test` | A test event from the dashboard or `test-webhook` | sample data |
| `billing.subscription_created` | A plan subscription started | `{ subscription }` |
| `billing.subscription_updated` | Plan, interval, extra accounts, status, or scheduled cancellation changed | `{ subscription, previous }` |
| `billing.subscription_canceled` | The subscription ended; the workspace is back on Free | `{ subscription }` |
| `billing.invoice_paid` | An invoice was paid | `{ invoice }` |
| `billing.payment_failed` | An invoice payment failed | `{ invoice }` |
| `billing.credits_purchased` | Prepaid credits were added | `{ credits: { amount_cents, balance_cents, pack_id, source } }` |
| `billing.accounts_over_limit` | Connected accounts exceed the plan; publishing is paused for the accounts beyond it | `{ accounts: { used, limit, paused_account_ids } }` |

Support, member, and billing events belong to no profile, so endpoints with `profile_ids` never receive them.

## Typical subscriptions

| Goal | Events |
| --- | --- |
| Show publishing status in your app | `post.published`, `post.partially_published`, `post.failed`, `posting.failed` |
| Approval flow | `post.approval_requested`, `post.approved`, `post.rejected` |
| Account health alerts | `account.disconnected`, `account.needs_reconnect` |
| Feed a helpdesk | `message.received`, `comment.received`, `review.received`, `mention.received` |
| CRM sync | `contact.created`, `contact.updated`, `contact.tag_added`, `contact.field_changed` |
| Lead capture | `lead.received` |
| Refresh dashboards | `analytics.synced` |

## Test events

`test-webhook` sends one signed request with `test: true` and the chosen `type` (default `webhook.test`). Its `data` is a placeholder (`{ "object": "sample", "message": "...", "type": "..." }`), not the payload of that type. It checks that the endpoint is reachable and verifies signatures; use the tables above to build handlers.
