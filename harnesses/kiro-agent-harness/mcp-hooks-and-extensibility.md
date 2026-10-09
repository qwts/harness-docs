# MCP, hooks, and extensibility

## MCP

- **✅** Local (stdio) and remote (HTTP/SSE) servers on IDE/CLI/Web (not Mobile). Server-provided prompts + resources surface via `#` mentions; elicitation requests supported.

### Configuration

- **✅** JSON config under `mcpServers` (workspace + user scopes — palette commands: "Kiro: Open workspace MCP config (JSON)" / "Kiro: Open user MCP config (JSON)"):

```json
{ "mcpServers": { "fetch": { "command": "uvx", "args": ["mcp-server-fetch"], "disabled": false } } }
```

- **⚠️** Paths are `.kiro/settings/mcp.json` (workspace) and `~/.kiro/settings/mcp.json` (user) per the configuration reference; the palette commands are the documented way to open them.
- **✅** Per-agent MCP: agent config `mcpServers` + `includeMcpJson` (false = agent-scoped servers only).
- **✅** `kiro://` one-click install links → confirmation dialog shows command + env/header names (values hidden); nothing is written until confirmed.
- **✅** Tool validation: name ≤64 chars incl. prefix, `^[a-zA-Z][a-zA-Z0-9_]*$`, non-empty description — failing tools are **excluded**. Descriptions >10,000 chars trigger a performance warning.
- Logs: Kiro panel → Output → "Kiro - MCP Logs".

## Hooks

- **✅** Standalone JSON files at `.kiro/hooks/*.json` (any filename; kebab-case recommended), auto-activated at session start. Format since IDE 1.0 / CLI 3.0.

### Schema (v1)

```json
{
  "version": "v1",
  "hooks": [{
    "name": "Lint on save",
    "description": "...",
    "trigger": "PostFileSave",
    "matcher": "\\.(ts|tsx)$",
    "action": { "type": "command", "command": "npx eslint --fix" },
    "timeout": 60,
    "enabled": true
  }]
}
```

| Field | Notes |
|---|---|
| `trigger` | PascalCase event name (`PostFileSave`, `PreToolUse`, `PostToolUse`, `Stop`, …) |
| `matcher` | optional regex — tool name for tool triggers, file path for file triggers; default match-all |
| `action.type` | `"command"` (shell in project root; gets session context as JSON on **STDIN**) or `"agent"` (inject prompt into conversation) |
| `timeout` | seconds, default 60; `0` disables; ignored for agent actions |
| `enabled` | default `true` |

### Confirmation prompts (`Stop` hooks)

- **✅** `confirm: { question, options: [{id, label, run}] }` gates a command hook.
- **✅** `confirmCommand` runs first; stdout JSON either `{ "skip": true }` or a replacement `{question, options}`; non-zero exit/invalid JSON → static fallback.

### Migration

- **✅** IDE 0.x → 1.0 moved hooks to standalone JSON w/ PascalCase triggers; CLI 2.x → 3.0 moved them out of embedded agent config (`kiro-cli agent migrate` auto-converts).

## Other extension points

- **Skills** — reusable instruction packages (feature-level listing)
- **Powers** — tools with built-in knowledge that activate on demand
- **Sub-agents** — parallel focused agents
- **ACP integrations** — use Kiro's agent from other editors/clients
