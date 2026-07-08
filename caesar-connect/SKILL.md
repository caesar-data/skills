---
name: caesar-connect
description: "Connect the Caesar remote MCP server to an MCP-capable host (Claude Code, Claude/ChatGPT web, Cursor, Windsurf, Codex). Use when the user wants Caesar web search inside a host app without installing the CLI, or asks to 'connect Caesar', 'add the Caesar MCP server', or 'set up the Caesar connector'. Prints the exact connector config; the host drives the OAuth login."
user-invocable: true
argument-hint: "[claude-code|claude|chatgpt|cursor|windsurf|codex|curl]"
compatibility: Requires an MCP-capable host. OAuth hosts need no API key; others use CAESAR_API_KEY.
license: MIT
metadata:
  author: caesar
---

# Connect Caesar to an MCP host

Caesar's remote MCP server gives any MCP host `web_search` and `web_fetch`
tools. Canonical endpoint: `https://mcp.trycaesar.com/mcp`
(`https://alpha.api.trycaesar.com/mcp` remains as a compatible alias). Two
ways to authenticate:

1. **OAuth (preferred, no key handling)** — hosts that implement MCP
   authorization discover the login from the server automatically: the user
   clicks Connect, approves in the browser, done. Never ask for an API key
   first if the host supports OAuth.
2. **Bearer API key** — for hosts without OAuth support and for CI: send
   `Authorization: Bearer $CAESAR_API_KEY`.

Emit exactly the block the user's host needs. If the host was not named, ask
which one they use.

## Claude Code

```bash
claude mcp add --transport http caesar https://mcp.trycaesar.com/mcp
```

Claude Code detects that the server requires authorization and opens the
browser to approve (`/mcp` inside a session shows the connection). To use a
raw key instead:

```bash
claude mcp add --transport http caesar https://mcp.trycaesar.com/mcp \
  --header "Authorization: Bearer $CAESAR_API_KEY"
```

## Claude (web/desktop) and ChatGPT

Add a custom connector with URL `https://mcp.trycaesar.com/mcp`. The host
walks the user through the browser approval; no key is entered.

## Cursor / Windsurf

```json
{
  "mcpServers": {
    "caesar": {
      "url": "https://mcp.trycaesar.com/mcp"
    }
  }
}
```

OAuth-capable versions prompt the user to authorize on first use. For older
versions, add `"headers": { "Authorization": "Bearer $CAESAR_API_KEY" }` —
and remind the user that most clients do not expand env vars in config
files, so the literal key value may need to be injected by their tooling.

## Codex

```toml
[mcp_servers.caesar]
url = "https://mcp.trycaesar.com/mcp"
bearer_token_env_var = "CAESAR_API_KEY"
```

## Raw JSON-RPC (verification)

```bash
curl -s https://mcp.trycaesar.com/mcp \
  -H 'Authorization: Bearer <credential>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Verify and troubleshoot

- Connected = the host lists `web_search` and `web_fetch`. Run a one-result
  search to confirm.
- HTTP 401 with a `WWW-Authenticate` header means the host has not completed
  the OAuth approval yet — re-run the host's connect/authorize step.
- HTTP 401 `invalid_api_key` on the Bearer path means the key is wrong or
  revoked — mint one in the console or run `caesar-search auth login`.
- Scope the tool set with `?tools=web_search` or `?profile=web-search` on
  the endpoint URL when the host should only search, not fetch.
- Never place credentials in the URL (query or path); header or OAuth only.
