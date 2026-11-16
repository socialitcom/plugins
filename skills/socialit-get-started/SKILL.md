---
name: socialit-get-started
description: This skill should be used when the user asks to "set up Socialit", "connect Socialit", "get started with Socialit", "is Socialit connected", "connect my Instagram to Socialit", "which accounts are connected", "create my first post", or has just installed the Socialit plugin. Walks through connecting the hosted MCP server with OAuth, confirming the workspace and role, checking connected social accounts and their health, connecting a network when there is none, and creating a first draft post without publishing it.
---

# Get started with Socialit

Use this flow the first time a user works with Socialit in this client, or whenever the Socialit tools are missing or failing to authenticate. Keep each step short and tell the user what you found before moving on.

## 1. Check the connection

Call `get-current-workspace`.

- **It works**: say "You're connected to the <name> workspace (<id>) as <role>, with <permissions> access." and go to step 2.
- **The tool does not exist**: the MCP server is not connected. Give the user the steps for their client from `references/connect.md`, then stop until they confirm.
- **401 or an auth error**: the sign in expired or was revoked. Ask the user to reconnect (most clients have a Reconnect or Authenticate button next to the server).

With OAuth, the user chose which workspaces this connection may use: all of them, or a selected set. When `can_switch_workspace` is `true`, call `list-workspaces` and show them with the user's role in each. To act in another one, pass `workspace` to the tool call ("In Acme, list my accounts"), or call `switch-workspace` to change the default. A workspace that is missing needs a reconnect, choosing it on the consent page.

If `permissions` has no `write`, the user can read but not create posts or connect accounts (a `viewer` role, a `read` key, or a client granted read only). Say so before step 3.

## 2. Check the connected accounts

Call `list-accounts`, then `get-accounts-health`.

Show a short table: network (`platform`), name or username, profile, and health. From the health result:

| Field | Meaning |
| --- | --- |
| `status` | `healthy`, `warning`, or `error` |
| `needs_reconnect` | The network revoked or expired the token. Posts to this account fail until it is reconnected |
| `can_post`, `can_fetch_analytics` | What the account can do with the permissions it granted |
| `issues`, `recommendations` | Plain language reasons and fixes |

`get-accounts-health` reads the stored state. For one account the user is worried about, `get-account-health` probes the token at the network live.

- **Healthy accounts**: go to step 4.
- **An account needs reconnecting**: offer a reconnect link (step 3 with `reconnect_account_id`), or the dashboard. Reconnecting keeps the account, its profile, and its history.
- **No accounts**: go to step 3.

## 3. Connect a network

Ask which network to connect. Two ways:

**In the app (simplest)**: the user opens https://app.socialit.com, goes to **Accounts**, and chooses **Connect account**. Recommend this for Bluesky, which uses an app password that must not be pasted into the chat.

**With a connect link from here**:

1. Optional: `list-connect-platforms` shows each network's connect method and what it requires. `list-profiles` shows the profiles; without `profile_id`, the account joins the workspace default profile.
2. Call `get-connect-url` with `platform` (such as `linkedin`, `instagram`, `youtube`), and `profile_id` if the user picked one. Mastodon also needs `instance` (for example `mastodon.social`). To repair an account, pass `reconnect_account_id: "sa_..."` instead.
3. Give the user the returned `url`. It opens the network's sign in; logins with several Pages, organizations, locations, or boards get a Socialit picker; the last page says the window can be closed. The link expires at `expires_at`.
4. When the user says they are done, call `list-accounts` again and confirm the new account.

Each new account counts against the plan's account allowance (3 on Free); reconnects never do. `account_limit_reached` means the plan is full: suggest an upgrade or disconnecting an unused account in the app.

## 4. Create a first draft post

Never publish or schedule in this step. Ask what the post should say and which accounts it should go to, then propose it:

```
Accounts: LinkedIn (Northwind), X (@northwind)
Text:     We just connected Socialit. More soon.
Status:   draft (nothing is published)
```

After the user agrees, call `create-post` with `content`, `account_ids` (the `sa_` IDs from `list-accounts`), and `draft: true`. Always send `draft: true` explicitly.

Report:

- The post ID (`pst_...`) and `status: draft`, and one posting per account (`get-post` shows them again later).
- Any `warnings` in the response. Drafts save even when a network would reject them (text too long for X, missing media for Instagram) or a network is not available for publishing yet; the warnings say which. Fix them before scheduling.
- That nothing went out, and the user can review it under **Posts** in the app.

`validate-post` checks the draft against every target (length, media, options) if the user wants to know it will pass. To schedule or publish it, the user must say so; then load `socialit-publishing` and use `schedule-post`.

## 5. Suggest next steps

Offer two or three, based on what the user wants:

- Schedule posts, fill queues, add media, and fix failed posts: `socialit-publishing`.
- Answer DMs, comments, reviews, and mentions: `socialit-inbox`.
- See what performed best and when to post: `socialit-analytics`.
- Contacts, broadcasts, sequences, and workflows: `socialit-automations`.
- Ads and boosted posts: `socialit-ads`.
- Get events in their own app: `socialit-webhooks`.
- Call the API from their code: `socialit-api`.

## Access levels

The AI client asks for permissions when it connects (all of `read`, `write`, and `admin` unless it asks for fewer). The consent page lists them, and the user chooses the workspaces there. In each workspace the connection acts with the user's role:

- `owner` and `admin`: everything, including API keys, webhooks, profiles, and publishing settings.
- `member`: posts, media, inbox, contacts, and automations; no workspace administration. With approvals on, a member's scheduled posts wait in `pending_approval`.
- `viewer`: read only.
- `billing`: billing and usage only.

Members and viewers can be limited to some profiles; they see only those accounts.

## API key option

Clients that cannot open a browser sign in (CI jobs, servers, the xAI API) can send a Socialit API key as `Authorization: Bearer <key>` to `https://api.socialit.com/mcp`. A workspace admin creates the key in the app under **Manage > API keys**. Use a `read` key for reporting agents, and restrict a key to the profiles an agent needs. Keep the key in an environment variable or the client's secret store. Never ask the user to paste it into the chat. Details are in `references/connect.md`.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| No Socialit tools | Server not added, or plugin not enabled | Follow `references/connect.md` |
| 401 on every call | Sign in expired or connection revoked | Reconnect the server |
| Fewer tools than expected | Toolsets trimmed, read only mode, a `read` key, or the user's role | Check the server URL and headers, the key, and the role in `get-current-workspace` |
| A workspace is missing | Not selected on the consent page | Reconnect and select it, or allow all workspaces |
| `admin_role_required` or `insufficient_permission` | The user's role does not allow it | A workspace admin does it or changes the role |
| `account_limit_reached` when connecting | The plan's account allowance is full | Upgrade, or disconnect an unused account |
| Account shows `needs_reconnect` | The network revoked or expired the token | `get-connect-url` with `reconnect_account_id`, or reconnect in the app |
| `403 missing_scopes` on a feature | The account granted fewer network permissions | Open the `reconnect_url` from the error |
| The draft comes back with `warnings` | A target network would reject it, or cannot publish yet | The draft is saved; fix the content or targets with `update-post` before scheduling |
