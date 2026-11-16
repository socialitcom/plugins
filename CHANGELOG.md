# Changelog

## 1.0.1

- Clients that only start local servers use the new `@socialit/mcp` npm package (`npx -y @socialit/mcp` with `SOCIALIT_API_KEY`), a bridge to the hosted server, instead of `mcp-remote`.
- The official MCP Registry entry is now `io.github.socialitcom/mcp`, published from github.com/socialitcom/mcp with the npm package and the hosted server. The plugin no longer carries `server.json`, and `com.socialit/socialit` is not used.

## 1.0.0

First release, for Socialit v1.

- Hosted Socialit MCP server at `https://api.socialit.com/mcp` with OAuth 2.1 sign in (`read`, `write`, and `admin` permissions, workspaces chosen on the consent page) or an API key, more than 670 tools with MCP annotations, toolset and read only filters, and opt-in Focus mode toolsets per network.
- Skills: `socialit`, `socialit-get-started`, `socialit-publishing`, `socialit-inbox`, `socialit-analytics`, `socialit-automations`, `socialit-ads`, `socialit-webhooks`, `socialit-api`.
- Manifests for ChatGPT and Codex (Agent Plugins), Cursor, Claude Code, and Grok.
- Cursor rule that keeps Socialit API keys and webhook secrets out of code.
