# Agents, sub-agents, and permissions

## Custom agents

- **✅** Custom agents live at `.kiro/agents/<name>.json` **or** `.kiro/agents/<name>.md` — equivalent JSON and Markdown formats per the custom-agent docs. Documented fields:

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

- **✅** Permissions are a first-class feature ("control what the agent can access") and surface-specific in behavior. Hooks gate tool calls via `PreToolUse`: a command hook can **block** the invocation (exit code 2 blocks intentionally; other non-zero exits are hook failures, not blocks) and a successful command's **stdout is added to agent context** — it cannot modify the pending tool call's arguments. Matcher is a regex on the tool name.
- **✅** `Stop`-trigger command hooks can require confirmation via a `confirm` block (static `question` + `options[{id,label,run}]`, or dynamic via `confirmCommand` stdout JSON).
- **⚠️** `.kiroignore` controls file visibility to the agent — but only on **IDE and CLI V3**; it is not enforced on Web/Mobile, so don't treat it as a secrets boundary in cloud sessions.
- ❓ Granular per-tool permission configuration (allow/deny lists per surface) is referenced in docs navigation but was not fully verified for this entry.

## Modes / workflow surfaces

- **✅** Specs (requirements → design → tasks) and Bugfix Specs are the planned-workflow modes; chat/vibe sessions are the unplanned mode. Both sit on the same harness.
- **✅** Checkpoints and rewind let you undo agent changes or fork a conversation.
