# MCP, hooks, and plugins

**Verified:** 2026-10-29

## MCP configuration

For the CLI, MCP settings live at:

```text
~/.cline/data/settings/cline_mcp_settings.json
```

`CLINE_MCP_SETTINGS_PATH` can relocate that file. Configuration uses an `mcpServers` map. Local stdio servers use `command` and `args`; remote servers use `url`.

```json
{
  "mcpServers": {
    "remote-server": {
      "type": "streamableHttp",
      "url": "https://example.com/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Use `"type": "streamableHttp"` explicitly for the recommended remote transport. Omitting `type` defaults to legacy `sse` for backward compatibility; omission does not mean “automatically choose the current transport.”

Servers support fields and operations including `disabled`, `autoApprove`, environment variables, headers, enable/disable controls, and OAuth authorization through `cline mcp`. Keep tokens out of committed examples and repositories.

❓ **UNVERIFIED:** This research did not establish a current project-local MCP settings file. Do not invent `.cline/mcp.json` or assume compatibility with another harness’s project MCP filename.

## File hooks are surface-specific

File hooks and SDK plugin hooks are different extension mechanisms. File-hook locations and filenames must be checked against the host surface.

| Surface | Workspace hooks | Global hooks | Filename convention |
|---|---|---|---|
| CLI | `.cline/hooks/` | `~/.cline/hooks/` | CLI examples use event files such as `PreToolUse.sh` |
| VS Code extension | `.clinerules/hooks/` | `~/Documents/Cline/Hooks/` | Exact event name, no extension, executable, with shebang |

`CLINE_HOOKS_DIR` and `--hooks-dir` add discovery locations. Do not replace the surface’s normal layout with an override assumption; check the effective CLI configuration when auditing a job.

A hook can cancel an operation with JSON such as:

```json
{
  "cancel": true,
  "errorMessage": "Blocked run_commands: command policy denied this operation."
}
```

CLI file hooks are disabled in `--yolo` mode. Therefore, a command policy implemented only as a file hook is not enforced under that bypass mode.

Hooks are executable artifacts. Review their source, interpreter, permissions, environment access, and failure behavior before enabling them.

## Plugins

Current plugin support applies to **SDK, CLI, and Kanban**, not the VS Code or JetBrains extensions.

| Scope | Location |
|---|---|
| Global plugins | `~/.cline/plugins/` |
| Project plugins | `.cline/plugins/` |
| CLI-managed installs | `~/.cline/plugins/_installed/` |

Installation sources can be a file URL, Git repository, npm package, or local path. Package plugins use a `cline` field in `package.json`, not a root `plugin.json` manifest:

```json
{
  "name": "my-cline-plugin",
  "version": "1.0.0",
  "cline": {
    "plugins": [
      {
        "paths": ["./index.ts"],
        "capabilities": ["tools", "hooks"]
      }
    ]
  }
}
```

Entry files are `.ts` or `.js` modules exporting `AgentPlugin`. Package plugins can bundle top-level `skills/`.

### Plugin lifecycle

Plugin hooks are:

```text
beforeRun
afterRun
beforeModel
afterModel
beforeTool
afterTool
onEvent
```

`beforeTool` can skip a call, replace input, require approval, append context, or stop the run. This makes it a policy enforcement point, but only when the plugin is actually loaded and the surrounding execution mode has not bypassed the relevant mechanism.

- Plugins normally run in per-session worker threads.
- Worker isolation provides crash isolation, **not a security boundary**.
- Plugins bind at session start; reloads affect new sessions, while running sessions retain their original code.
- Default `hookErrorMode` is `"ignore"`; `"throw"` fails the run.
- Plugin hook timeout is 3 seconds; repeated failures can degrade and then disable a plugin.

A fail-open hook error mode and automatic degradation are important audit considerations. If a plugin implements a required control, explicitly evaluate whether `"ignore"` is acceptable and monitor whether the plugin remains active.

## Sources

- [MCP overview](https://docs.cline.bot/mcp/mcp-overview)
- [CLI reference](https://docs.cline.bot/cli/cli-reference)
- [Hooks](https://docs.cline.bot/customization/hooks)
- [Plugins](https://docs.cline.bot/customization/plugins)
- [SDK plugins](https://docs.cline.bot/sdk/plugins)
- [Writing plugins](https://docs.cline.bot/sdk/guides/writing-plugins)
- [`.clineignore`](https://docs.cline.bot/customization/clineignore)
