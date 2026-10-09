# Agents, sub-agents, and permissions

## Custom agents

- **✅** Kiro supports custom agents defined by JSON config files (agent config). Documented fields from official docs:

```json
{
  "name": "myagent",
  "description": "My special agent",
  "mcpServers": {
    "fetch": { "command": "fetch3.1", "args": [] }
  },
  "includeMcpJson": false,
  "resources": ["file://.kiro/steering/**/*.md"]
}
```

| Field | Meaning |
|---|---|
| `name` | agent identifier |
| `description` | purpose/usage |
| `mcpServers` | MCP servers this agent may use, in the same shape as `mcp.json` |
| `includeMcpJson` | `true` → agent also sees servers from user + workspace MCP config; `false` → only its own `mcpServers` |
| `resources` | file URIs pulled in as steering/context |

- **✅** Sub-agents exist as a feature: "delegate parallel tasks to focused sub-agents" (capability listing; deeper schema not yet documented in the pages verified).
- ⚠️ The docs list "primary-agent selection" as a surface-specific behavior — the same agent set can resolve differently per surface.

## Permissions

- **✅** Permissions are a first-class feature ("control what the agent can access") and surface-specific in behavior. The hooks system can gate tool calls: a `PreToolUse` hook can block or modify tool execution based on the tool name matched via regex `matcher`.
- **✅** `Stop`-trigger command hooks can require confirmation via a `confirm` block (static `question` + `options[{id,label,run}]`, or dynamic via `confirmCommand` stdout JSON).
- **✅** `.kiroignore` controls file visibility to the agent — the data-access permission layer.
- ❓ Granular per-tool permission configuration (allow/deny lists per surface) is referenced in docs navigation but was not fully verified for this entry.

## Modes / workflow surfaces

- **✅** Specs (requirements → design → tasks) and Bugfix Specs are the planned-workflow modes; chat/vibe sessions are the unplanned mode. Both sit on the same harness.
- **✅** Checkpoints and rewind let you undo agent changes or fork a conversation.
