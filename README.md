# Socialit plugin for ChatGPT, Codex, Claude, Cursor, and Grok

The official [Socialit](https://socialit.com) plugin. It connects your AI assistant to the hosted Socialit MCP server, so it can draft, schedule, and publish posts to your connected social accounts, answer direct messages, comments, reviews, and mentions, read analytics, run contacts, broadcasts, sequences, and workflows, manage ads, and configure webhooks. It also ships skills that teach the assistant how to use Socialit well and how to integrate the Socialit API into your code.

You sign in with your Socialit account through OAuth and choose which workspaces the plugin can use. Revoke it at any time under **Account > Connected apps** in Socialit. No API key is stored in the plugin.

## What is included

| Part | Description |
| --- | --- |
| MCP server | `https://api.socialit.com/mcp`, more than 670 tools across posts, media, queues, accounts, the inbox, analytics, contacts, broadcasts, sequences, workflows, comment automations, ads, webhooks, API keys, and the team, plus opt-in Focus mode toolsets per network |
| `socialit` skill | Choosing MCP or REST, workspaces and permissions, safe publishing, IDs, errors, and the full tool list |
| `socialit-get-started` skill | Connect, confirm the workspace, check the connected accounts, and create a first draft post |
| `socialit-publishing` skill | Drafts, scheduling, queues, media, per-network options, approvals, and failed posts |
| `socialit-inbox` skill | Direct messages, comments, reviews, mentions, and comment automations |
| `socialit-analytics` skill | Post, account, and inbox analytics, best times to post, and reports |
| `socialit-automations` skill | Contacts, custom fields, broadcasts, sequences, and workflows |
| `socialit-ads` skill | Ad accounts, campaigns, audiences, insights, and lead forms, with spend confirmed first |
| `socialit-webhooks` skill | Events, endpoints, signature verification, retries, and debugging deliveries |
| `socialit-api` skill | The REST API in your code: authentication, pagination, idempotency, rate limits, media uploads |
| Cursor rule | Keeps Socialit API keys and webhook secrets out of source code |

## Install

### Claude Code

```bash
claude plugin marketplace add socialitcom/plugins
claude plugin install socialit@socialit
```

Then run `/mcp`, select **socialit**, and choose **Authenticate**.

### Claude

On claude.ai, Claude Desktop, or mobile: **Settings > Connectors > Add custom connector**. Name it `Socialit`, enter `https://api.socialit.com/mcp`, choose **Connect**, and sign in.

### Codex

```bash
codex plugin marketplace add socialitcom/plugins
```

Open `/plugins`, install **Socialit**, and sign in when prompted.

### ChatGPT

Turn on **Developer mode** under **Settings > Apps & Connectors > Advanced**, choose **Create**, name the connector `Socialit`, enter `https://api.socialit.com/mcp` with OAuth authentication, and sign in.

### Cursor

Add Socialit to `~/.cursor/mcp.json`. Cursor opens the Socialit sign in the first time a tool is used.

```json
{
  "mcpServers": {
    "socialit": {
      "url": "https://api.socialit.com/mcp"
    }
  }
}
```

### Grok

```bash
grok plugin marketplace add socialitcom/plugins
grok plugin install socialit --trust
```

On grok.com, open **Connectors > New Connector > Custom** and enter `https://api.socialit.com/mcp`.

### Other clients

Any client that supports remote MCP servers over Streamable HTTP works the same way: give it `https://api.socialit.com/mcp` and sign in. Clients that only start local (stdio) servers, such as Claude Desktop's config file, use the [`@socialit/mcp`](https://www.npmjs.com/package/@socialit/mcp) package with a Socialit API key: the command `npx -y @socialit/mcp` with `SOCIALIT_API_KEY` in its environment. It bridges to the hosted server, so the tools are the same ([socialitcom/mcp](https://github.com/socialitcom/mcp)).

## Access and security

- **OAuth 2.1 with PKCE.** Your assistant never sees your password. The consent page shows the app and the permissions it asks for: `read` (posts, accounts, inbox, analytics), `write` (create and change posts, reply in the inbox, upload media), and `admin` (workspace administration such as API keys and webhooks). You choose which workspaces it can use, and it acts with your role and profile access in each.
- **Confirmation before impact.** Tools that publish, send messages, spend money, delete data, or call external URLs carry MCP annotations that let your client ask you first, and the skills tell the assistant to confirm and to prefer drafts. The Socialit server does not enforce the confirmation itself; whether you are asked depends on your client and its settings.
- **Revoke at any time** in Socialit under **Account > Connected apps**. Every tool call appears in the workspace request logs and activity with the client's name.
- **API keys stay with Socialit.** The MCP server signs in with OAuth and needs no key. Only when it isn't connected do the skills use a Socialit API key from your environment (`SOCIALIT_API_KEY`), and only to call Socialit's own REST API (listed below). Code the API skill writes into your project reads the key the same way.
- **Headless clients** (CI jobs, servers, the xAI API) can send a Socialit API key as `Authorization: Bearer <key>` to the same server. Use a `read` key for reporting agents, and restrict a key to the profiles an agent needs.

To expose fewer tools, add `?toolsets=posts,media,accounts` or `?read_only=true` to the server URL, or send the `X-MCP-Toolsets` and `X-MCP-Readonly` headers.

## Endpoints this plugin uses

The plugin talks only to Socialit:

| Endpoint | Purpose |
| --- | --- |
| `https://api.socialit.com/mcp` | Hosted MCP server (Streamable HTTP) |
| `https://api.socialit.com/.well-known/oauth-protected-resource/mcp` | OAuth resource metadata |
| `https://api.socialit.com/.well-known/oauth-authorization-server` | OAuth server metadata |
| `https://api.socialit.com/oauth/register` | Dynamic client registration |
| `https://api.socialit.com/oauth/authorize` | Sign in and consent |
| `https://api.socialit.com/oauth/token` | Token exchange and refresh |
| `https://api.socialit.com/oauth/revoke` | Token revocation |
| `https://api.socialit.com/v1` | REST API, used by code that the API skill writes into your project |

## Requirements

A Socialit account and at least one connected social account. The `socialit-get-started` skill walks you through connecting one.

## Development

This repository is published from Socialit's main repository, and each release replaces its contents. `skills/socialit/references/tools.md` is generated from the Socialit MCP server catalog. Please report problems and suggestions as issues rather than pull requests, or write to support@socialit.com.

| File | Read by |
| --- | --- |
| `plugin.json`, `mcp.json` | ChatGPT, Codex, and other Agent Plugins clients |
| `.cursor-plugin/` | Cursor |
| `.claude-plugin/`, `.mcp.json` | Claude Code, Claude directory, Grok |
| `.grok-plugin/plugin.json` | Grok |
| `.agents/plugins/marketplace.json` | Codex marketplace |

## Support

[socialit.com/contact](https://socialit.com/contact/) or support@socialit.com. Documentation: the [hosted MCP server guide](https://socialit.com/docs/mcp/) and the [API reference](https://socialit.com/docs/api-reference/). Privacy policy: [socialit.com/privacy](https://socialit.com/privacy/).

## License

MIT
