# MCP and hooks

**Verified:** 2026-10-29

## MCP configuration files

CLI and shared customization paths include:

```text
Global:    ~/.gemini/config/mcp_config.json
Workspace: .agents/mcp_config.json
```

The App also exposes MCP Store installation and management through Settings > Customizations. Use unique server names across scopes; CLI documents that when duplicate names occur, the already-running server keeps the name.

## Schema and transports

The root object contains an `mcpServers` map. Each server uses one primary transport property:

```json
{
  "mcpServers": {
    "local": {
      "command": "node",
      "args": ["server.js"]
    },
    "remote": {
      "serverUrl": "https://example.com/mcp"
    }
  }
}
```

`command` configures stdio. `serverUrl` configures remote Streamable HTTP; `url` is also accepted. Optional fields include `args`, `env`, `cwd`, `headers`, `authProviderType`, `oauth`, `disabled`, `enabledTools`, `disabledTools`, and `timeoutSeconds`.

Default timeouts are:

- 300 seconds to connect and list tools;
- 180 seconds per tool call.

A positive `timeoutSeconds` replaces both limits; a negative value disables both; zero or omission preserves defaults.

## Authentication and permissions

Remote authentication options include HTTP headers, Google Application Default Credentials through `authProviderType: "google_credentials"`, automatic OAuth with Dynamic Client Registration, and manual OAuth client credentials.

OAuth tokens are stored at:

```text
~/.gemini/antigravity/mcp_oauth_tokens.json
```

Unconfigured MCP tools default to Ask. Permission targets include `mcp(server/tool)`, `mcp(server/*)`, and `mcp(*)`. Tool exposure filters and execution permission are separate: `enabledTools` controls what the agent sees, while permission rules control whether calls require approval.

## Hooks

Hook files:

```text
Workspace: .agents/hooks.json
Global:    ~/.gemini/config/hooks.json
```

The CLI does not load hooks from a `hooks` key in `settings.json`, even though that key can be retained as unknown data.

Documented events:

| Event | Timing | Matcher |
|---|---|---|
| `SessionStart` | Once before first model invocation | Ignored |
| `PreToolUse` | Before tool execution | Tool-name regex |
| `PostToolUse` | After tool completion | Tool-name regex |
| `PreInvocation` | Before model call | Ignored |
| `PostInvocation` | After model invocation | Ignored |
| `Stop` | When execution terminates | Ignored |

The file maps hook names to event configurations. Hook definitions can be disabled with `enabled: false`. Tool matchers accept `*` or an empty string for all tools, exact names, alternation, and regular expressions.

Example:

```json
{
  "my-linter-hook": {
    "PostToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/lint.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

Hooks receive session metadata and event payloads on stdin. `PreToolUse` can return decisions including `allow`, `auto_approve`, `deny`, `ask`, `force_ask`, and `deny_unless_prior_grant`. It can also return `overwrite` for a shallow merge into tool arguments and `permissionOverrides` for resource-specific policy changes. `PostToolUse` returns an empty JSON object in the documented contract.

These capabilities are Antigravity's schema; similarly named hooks in Cursor, Copilot, or Muse are not drop-in equivalents.

## Sources

- [MCP](https://antigravity.google/docs/mcp/)
- [Hooks](https://antigravity.google/docs/hooks/)
- [CLI settings](https://antigravity.google/docs/cli/settings/)
- [Permissions](https://antigravity.google/docs/permissions/)
