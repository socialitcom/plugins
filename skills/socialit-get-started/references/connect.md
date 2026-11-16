# Connect the Socialit MCP server

Server URL: `https://api.socialit.com/mcp` (Streamable HTTP). Sign in uses OAuth 2.1 with PKCE and dynamic client registration. Clients discover it from `https://api.socialit.com/.well-known/oauth-protected-resource/mcp`, so most only need the URL. On the consent page the user signs in to Socialit, sees the permissions the client asks for (`read`, `write`, `admin`), and picks the workspaces it may use: all of them, or a selected set.

The server works on every plan and counts against the workspace's hourly API request budget.

## Claude Code

Install the plugin, which brings the server and the skills:

```bash
claude plugin marketplace add socialitcom/plugins
claude plugin install socialit@socialit
```

Or add only the server:

```bash
claude mcp add --transport http socialit https://api.socialit.com/mcp
```

Run `/mcp`, select **socialit**, and choose **Authenticate**.

## Claude (claude.ai, Desktop, and mobile)

1. Open **Settings > Connectors** and choose **Add custom connector**.
2. Name it `Socialit` and enter `https://api.socialit.com/mcp`.
3. Choose **Connect**, sign in to Socialit, choose the workspaces, and approve.
4. In a chat, turn on Socialit from the tools menu.

On Team and Enterprise plans an owner adds the connector for the organization first.

## ChatGPT

1. Open **Settings > Apps & Connectors > Advanced** and turn on **Developer mode** (on plans that support custom connectors).
2. Choose **Create**, name the connector `Socialit`, enter `https://api.socialit.com/mcp`, and pick OAuth authentication.
3. Sign in to Socialit, choose the workspaces, and approve.
4. In a chat, enable Socialit from the tools menu. ChatGPT asks before tools that change data.

While trying it out, `https://api.socialit.com/mcp?read_only=true` keeps it to read tools. Remove the parameter when the user wants ChatGPT to create posts.

## Cursor

Add Socialit to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "socialit": {
      "url": "https://api.socialit.com/mcp"
    }
  }
}
```

Or open this link to add it:

```
cursor://anysphere.cursor-deeplink/mcp/install?name=socialit&config=eyJ1cmwiOiJodHRwczovL2FwaS5zb2NpYWxpdC5jb20vbWNwIn0=
```

The `config` value is the base64 of `{"url":"https://api.socialit.com/mcp"}`. Cursor opens the Socialit sign in the first time a tool is used.

## Codex

```bash
codex plugin marketplace add socialitcom/plugins
```

Open `/plugins` in Codex, install **Socialit**, and sign in when prompted. To add only the server:

```bash
codex mcp add socialit --url https://api.socialit.com/mcp
codex mcp login socialit
```

## Grok

Grok Build CLI:

```bash
grok plugin marketplace add socialitcom/plugins
grok plugin install socialit --trust
```

Or add only the server: `grok mcp add --transport http socialit https://api.socialit.com/mcp`. Grok opens a browser to sign in.

grok.com: open **Connectors**, choose **New Connector > Custom**, and enter `https://api.socialit.com/mcp`.

## Other clients

Any client that supports remote MCP servers over Streamable HTTP works the same way: give it `https://api.socialit.com/mcp` and sign in. Clients that only start local (stdio) servers, such as Claude Desktop's config file, use the `@socialit/mcp` package with an API key (see "API key for headless clients" below for creating one). It bridges to the hosted server, so the tools are the same:

```json
{
  "mcpServers": {
    "socialit": {
      "command": "npx",
      "args": ["-y", "@socialit/mcp"],
      "env": { "SOCIALIT_API_KEY": "sk_live_..." }
    }
  }
}
```

`SOCIALIT_TOOLSETS` (for example `posts,media,accounts`) and `SOCIALIT_READ_ONLY=true` in the same `env` narrow the tools. It needs Node.js 20 or later.

## API key for headless clients

CI jobs, servers, and the xAI API cannot run a browser sign in. A workspace admin creates an API key in the Socialit app under **Manage > API keys** (`full` or `read` permission, optionally restricted to some profiles), stores it in an environment variable or secret store, and the client sends it as a bearer token:

```
Authorization: Bearer sk_live_...
```

Claude Code, reading the key from the environment:

```bash
claude mcp add --transport http socialit https://api.socialit.com/mcp \
  --header "Authorization: Bearer $SOCIALIT_API_KEY"
```

Cursor:

```json
{
  "mcpServers": {
    "socialit": {
      "url": "https://api.socialit.com/mcp",
      "headers": { "Authorization": "Bearer sk_live_..." }
    }
  }
}
```

Keep a file with a key out of shared repositories; prefer the global `~/.cursor/mcp.json`.

xAI API remote MCP tool:

```json
{
  "type": "mcp",
  "server_url": "https://api.socialit.com/mcp",
  "server_label": "socialit",
  "authorization": "<SOCIALIT_API_KEY>"
}
```

An API key connection is fixed to the key's workspace and has no `workspace` argument. Use a `read` key for reporting agents, and restrict a key to the profiles an agent needs.

## Trim the tool list

Add these to the server URL, or send them as headers, to expose fewer tools:

| Option | Query | Header | Example |
| --- | --- | --- | --- |
| Only some toolsets | `?toolsets=` | `X-MCP-Toolsets` | `posts,media,accounts,analytics` |
| Hide tools that change data | `?read_only=true` | `X-MCP-Readonly` | `true` |

Toolsets: `team`, `api_keys`, `webhooks`, `logs`, `accounts`, `posts`, `queues`, `media`, `templates`, `tools`, `inbox`, `analytics`, `contacts`, `custom_fields`, `broadcasts`, `sequences`, `workflows`, `comment_automations`, `ads_measurement`, `ads`, `platform_tools`, `focus`, `billing`, `support`. The `workspace` toolset is always included.

Focus mode typed toolsets are opt in, one per network: `focus_youtube`, `focus_x`, `focus_reddit`, `focus_instagram`, `focus_facebook`, `focus_linkedin`, `focus_tiktok`, `focus_threads`, `focus_bluesky`, `focus_pinterest`, `focus_google_business`, `focus_snapchat`, `focus_discord`, `focus_mastodon`. Select only the networks the agent works on, for example `?toolsets=focus,focus_youtube`.

## Disconnect

In the Socialit app, open **Account > Connected apps** and choose **Revoke access** next to the client. It loses access to every workspace at once. Every tool call also appears in the workspace request logs and activity with the client's name.
