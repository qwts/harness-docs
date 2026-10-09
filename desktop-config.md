# Claude Desktop — MCP Host Configuration (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

## Role

✅ TRUE — Claude Desktop is a graphical MCP **host**: at startup it reads one static JSON config file, spawns the defined MCP servers as child processes, and maintains persistent stdio pipelines to them. Tool calls made in the Desktop chat UI are routed through these child processes. Configuration in this file is treated as explicitly authorized — treat write access to it as high-privilege.

## Configuration file paths

| OS | Path | Verdict |
|---|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` | ✅ TRUE |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` | ✅ TRUE |
| Linux | `~/.config/Claude/claude_desktop_config.json` | ⚠️ PARTIAL — Anthropic ships Claude Desktop for **macOS and Windows only**. This path applies to unofficial/Electron-wrapped builds. On Linux, use Claude Code (CLI) for MCP. |

The file may not exist on a fresh install; create it manually or via Settings → Developer → Edit Config. Only this exact path is read; Desktop does not search elsewhere. ✅ TRUE (for Desktop).

## JSON schema

Root object must contain an `mcpServers` key mapping server alias → execution parameters. Strict JSON: trailing commas and mismatched brackets break parsing and silently disable all servers. ✅ TRUE

| Key | Requirement | Notes | Verdict |
|---|---|---|---|
| `command` | Required | Executable to launch. ⚠️ PARTIAL: absolute paths are a strong **recommendation** (GUI-launched processes may lack shell `PATH`), not a schema-enforced constraint. | ⚠️ |
| `args` | Optional | Array of string arguments passed sequentially. | ✅ TRUE |
| `env` | Optional | Object of string key→value environment variables injected into the child process. | ✅ TRUE |

Example:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "/usr/local/bin/npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/Desktop"]
    }
  }
}
```

## Windows execution constraint

✅ TRUE — On Windows, Node.js-based servers launched via `npx` commonly fail because `npx` is a batch wrapper (`npx.cmd`) spawned without a shell. Use one of:
- the absolute path to the `.cmd` variant (e.g. `C:\Program Files\nodejs\npx.cmd`), or
- a shell wrapper (`cmd /c npx ...`) approach.

Equivalent issues apply to other `.cmd`-shimmed package executables. On native Windows (not WSL), Claude Code stdio servers using `npx` need the same treatment.

## Lifecycle: config changes require a full restart

✅ TRUE — After editing the config file, Claude Desktop must be **completely terminated and relaunched**; it parses the file only at startup. Closing the window is insufficient because the process keeps running in the background:
- macOS: quit via `Cmd+Q` or Claude → Quit Claude.
- Windows: exit via the system tray icon or explicit process termination.

## Environment variable inheritance

✅ TRUE — Desktop inherits environment variables from the **GUI session**, not from shell profile scripts (`.bashrc`, `.zshrc`, etc.). A server whose API keys are defined only in a shell profile will work from a terminal but fail under Desktop. Mitigation for agents: inject required credentials directly into the `env` block of the Desktop config (mind plaintext-at-rest risk).

## Desktop ↔ CLI isolation

✅ TRUE — Claude Desktop and Claude Code maintain fully isolated configuration states. Claude Code never reads `claude_desktop_config.json`; edits to it have no effect on the CLI. Use `claude mcp add-from-claude-desktop` (macOS/WSL only) or manual translation to move servers across — see [cli-mcp-config.md](cli-mcp-config.md).

## Desktop extensions note (newer than the source draft)

Desktop also supports packaged **MCPB desktop extensions** and remote connectors via the UI, which are easier than hand-editing JSON for supported servers. The raw config file remains the programmatic injection point for local stdio servers.
