# Providers and MCP

## Provider architecture

- **✅** OpenCode routes models through the **Vercel AI SDK** + **Models.dev** catalog — 75+ providers supported, plus local models.
- **✅** Adding a provider = `/connect` in the TUI (stores creds) + optional `provider` config block.
- **✅** Credentials live in `~/.local/share/opencode/auth.json` — not in config files.
- **✅** Model IDs are `provider/model-id` (e.g. `anthropic/claude-sonnet-4-5`).

### `provider` config options

| Option | Effect |
|---|---|
| `options.baseURL` | Custom endpoint / proxy base URL |
| `blacklist` | Hide listed model IDs from `/models` picker |
| `whitelist` | Show only listed models (combine: whitelist narrows, blacklist removes) |

## First-party model services

- **✅** **OpenCode Zen** — curated, team-tested model list; `/connect` → opencode.ai/auth → API key. Optional, works like any provider.
- **✅** **OpenCode Go** — low-cost subscription for open coding models; same `/connect` flow.

## MCP servers

Defined under `mcp.<name>`; tools appear alongside built-ins (docs warn MCP tools eat context — GitHub MCP in particular).

### Local

```jsonc
"my-mcp": {
  "type": "local",
  "command": ["npx", "-y", "my-mcp-command"],
  "cwd": "...",            // workspace-relative
  "environment": {"K": "v"},
  "enabled": true,
  "timeout": 5000          // ms, default 5s
}
```

### Remote

```jsonc
"my-mcp": {
  "type": "remote",
  "url": "https://mcp.example.com/mcp",
  "enabled": true,
  "headers": {"Authorization": "Bearer {env:MY_API_KEY}"},
  "oauth": {},             // or false to disable; clientId/clientSecret/scope optional
  "timeout": 5000
}
```

### OAuth

- **✅** Automatic: 401 → OAuth flow; Dynamic Client Registration (RFC 7591) when supported; tokens stored in `~/.local/share/opencode/mcp-auth.json`.
- `oauth: false` disables auto-detection for API-key servers.
- Pre-registered creds: `oauth: { clientId, clientSecret, scope }` (use `{env:}` interpolation).

### MCP CLI

| Command | Purpose |
|---|---|
| `opencode mcp auth <name>` | Trigger OAuth flow (opens browser) |
| `opencode mcp list` / `mcp auth list` | Status of servers / OAuth state |
| `opencode mcp logout <name>` | Remove stored creds |
| `opencode mcp debug <name>` | Auth status + connectivity + discovery probe |

### Gating MCP tools

- **✅** MCP tools register as `<server>_<tool>`; gate via global `tools` map or per-agent `tools`/`permission` with globs (`"my-mcp*": false`).
- Pattern for per-agent-only MCP: disable globally in `tools`, re-enable inside `agent.<name>.tools`.
- Remote-config org MCPs (from `.well-known/opencode`) load as supplied; orgs *may* ship them `enabled: false` for opt-in — it is not forced.
