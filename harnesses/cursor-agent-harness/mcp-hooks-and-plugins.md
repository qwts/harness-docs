# MCP, hooks, and plugins

**Verified:** 2026-10-29

## MCP configuration

Cursor uses:

```text
Project: .cursor/mcp.json
User:    ~/.cursor/mcp.json
```

Supported transports include stdio, SSE, and Streamable HTTP. Cursor documents support for MCP tools, prompts, resources, roots, elicitation, and Apps. That is broader than a tools-only client and should not be generalized to other harnesses or to cloud subagents without checking their configuration source.

Project and user scope determine where configuration is shared; they do not make a local server definition automatically available in a cloud runtime.

## Hooks

Hook configuration lives at:

```text
Workspace: .cursor/hooks.json
User:      ~/.cursor/hooks.json
```

Local and cloud execution differ in important ways:

- Cloud hooks do not run during early read-only exploratory turns.
- For backend-hosted HTTP MCP tools, cloud `preToolUse` can deny a call but ignores `updated_input`.
- Cloud `postToolUse` receives redacted output for those backend-hosted HTTP MCP tools.

A local hook that rewrites tool arguments or expects complete output is therefore not automatically portable to cloud execution.

## Plugin formats

Cursor supports two plugin layouts:

| Format | Manifest |
|---|---|
| Agent Plugins | Root `plugin.json` |
| Cursor Plugins | `.cursor-plugin/plugin.json` |

Do not infer that every portable Agent Plugins feature is implemented identically. In particular, Cursor does not expand the portable standard's `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` placeholders in `mcp.json`. Use the Cursor-supported plugin-root variable:

```text
${CURSOR_PLUGIN_ROOT}
```

This is a concrete compatibility difference, not merely a manifest-location difference. Review skills, agents, hooks, MCP configuration, and variable expansion separately when porting a plugin.

## Sources

- [MCP](https://cursor.com/docs/mcp)
- [Hooks](https://cursor.com/docs/hooks)
- [Plugins](https://cursor.com/docs/plugins)
- [Subagents](https://cursor.com/docs/subagents)
