# Configuration and settings

**Verified:** 2026-11-16 (Pi v1.5.2)

## Agent directory

“User-level configuration lives in the agent directory, which defaults to `~/.pi/agent`” (`PI_CODING_AGENT_DIR` or the SDK's `agentDir` option relocates it):

| Path | Responsibility |
|---|---|
| `settings.json` | “User-level settings, including preferences, defaults, resource paths, and Pi package declarations.” |
| `keybindings.json` | Custom terminal UI and application keybindings. |
| `mcp.json` | “MCP servers available in every project.” |
| `models.json` | “Compatible endpoints, models, and model overrides.” |
| `auth.json` | “Saved API keys and OAuth credentials.” |
| `AGENTS.override.md`, `AGENTS.md`, `AGENTS.MD`, `CLAUDE.md`, or `CLAUDE.MD` | “User instructions applied across working directories.” |
| `SYSTEM.md` | “Replaces Pi's default system prompt.” |
| `APPEND_SYSTEM.md` | “Adds instructions to Pi's system prompt.” |
| `extensions/`, `skills/`, `prompts/`, `themes/` | User extensions, skills, prompt templates (“exposed as slash commands”), themes. |

## Project `.pi` directory

“Project configuration lives in `.pi` under the working directory and loads after project trust is granted. The only exception is `sessionDir`, which Pi reads before resolving trust so it can locate sessions.” The project tree mirrors the agent directory: `.pi/settings.json`, `.pi/mcp.json`, `.pi/SYSTEM.md`, `.pi/APPEND_SYSTEM.md`, `.pi/extensions/`, `.pi/skills/`, `.pi/prompts/`, `.pi/themes/`. For the system-prompt files, “the trusted project file takes precedence over the corresponding agent-directory file. Files with the same name are not combined.”

Precedence summary: “Project settings override agent-directory settings. Resource lists are combined.” Three settings are agent-directory-only: `defaultProjectTrust` (“**Can only be set in agent-directory settings**”), `httpProxy`, and `selfUpdateCommand`. `/settings` changes common preferences interactively; “Run `/reload` after manually changing settings, keybindings, instructions, or resources.”

## Context files

“Context files are separate from project `.pi` configuration. Pi loads them from the agent directory, the working directory, and its parent directories. A context file applies whenever Pi runs in its directory or anywhere below it.” Overrides are narrow: “An `AGENTS.override.md` replaces `AGENTS.md` or `CLAUDE.md` only in the same directory. It does not suppress context files from the agent directory or other directories.” And crucially: “Context-file discovery does not require project trust.” `-nc`/`--no-context-files` disables discovery.

## Settings reference highlights

- **Model and thinking:** `defaultProvider`, `defaultModel`, `defaultThinkingLevel` (`"medium"`), `modelThinkingLevels`, `thinkingBudgets`, `enabledModels`, `cacheWarming` (`"off" | "streaming" | "idle"`, default `"idle"` — “Cache warming runs only when the model declares a cache lifetime and Pi estimates at least $0.05 in avoided cache-miss cost”).
- **Interaction:** `steeringMode` / `followUpMode` (`"all" | "one-at-a-time"`, default `"one-at-a-time"`), `externalEditor`, `doubleEscapeAction` (`"tree" | "fork" | "none"`, default `"tree"`), `defaultProjectTrust` (`"ask"` default).
- **Tools:** `defaultTools` (default `read`, `bash`, `edit`, `write`; “Plain names replace the defaults; `+name` adds a tool and `-name` removes one”), `codemode.mode` (`"on" | "only"`), `codemode.inlineBudget` (3000).
- **Sessions and context:** `sessionDir`, `sessionShareDestination` (`"auto" | "gist" | "radius"`), `compaction.enabled` (true), `compaction.reserveTokens` (16384), `compaction.keepRecentTokens` (20000), `compaction.modelOverrides`, `branchSummary.reserveTokens` (16384), `branchSummary.skipPrompt`.
- **Network and retries:** `transport` (`auto`/`sse`/`websocket`/`websocket-cached`), `httpIdleTimeoutMs` (300000), `retry.enabled` (true), `retry.maxRetries` (3), `retry.baseDelayMs` (2000), `retry.maxAgentDelayMs` (60000), `retry.provider.maxRetries` (0 — “Keep `retry.provider.maxRetries` at `0` unless provider-level retries are required. Provider retries can delay Pi from handling quota and usage-limit errors itself”).
- **Shell:** `shellPath`, `shellCommandPrefix`, `npmCommand`, `selfUpdateCommand`.
- **Resources:** `packages`, `extensions`, `skills`, `prompts`, `themes` arrays (“Resource arrays support glob exclusions with `!pattern`, exact inclusion with `+path`, and exact exclusion with `-path`”), `enableSkillCommands` (true — “Register skills as `/skill:name` commands”). Built-in extensions are “named `builtin:mcp`, `builtin:llama.cpp`, `builtin:codemode`, and `builtin:tool-search` in `extensions`. They load by default; `-builtin:mcp` disables one.”
- **Updates, telemetry, warnings:** `collapseChangelog`, `enableInstallTelemetry` (default true — “anonymous install/update reporting and selected provider attribution headers. Does not control update checks”), `enableAnalytics` (default **false**), `warnings.anthropicExtraUsage` (true).

Add `"$schema": "https://pi.dev/schemas/settings.schema.json"` for editor validation.

## Keybindings and terminal

`<agent-dir>/keybindings.json` maps “each action identifier to one key or a list of keys” (`"app.session.new": "ctrl+shift+n"`); “A configured value replaces the default for that action. Use an empty list to disable an action's keybindings.” Modifiers are `ctrl`, `shift`, `alt`, `super` (“`super` bindings require a terminal that reports the modifier separately, typically through the Kitty keyboard protocol”). `/hotkeys` shows active shortcuts. TUI modes: “Fullscreen mode, the default, keeps the editor and status area fixed while the transcript scrolls within the terminal window. Regular mode uses the terminal's normal scrollback. Choose a mode through `/settings` or `--tui-mode`.”

## Environment variables

Process markers: “`AI_AGENT=pi` is a generic marker that lets tooling identify Pi as the agent that launched the process”, and “`PI_CODING_AGENT=true` is Pi-specific and lets child processes detect that they run inside Pi.” Shell-tool session injection — “Commands run by the `bash` and `powershell` tools receive the current Pi session state:” `PI_SESSION_ID`, `PI_SESSION_FILE` (“unset for ephemeral sessions”), `PI_PROVIDER`, `PI_MODEL`, `PI_REASONING_LEVEL`. “They are not injected into user-entered `!` or `!!` commands.”

Process configuration: `PI_CODING_AGENT_DIR`, `PI_CODING_AGENT_SESSION_DIR` (overridden by `--session-dir`), `PI_PACKAGE_DIR`, `PI_OFFLINE` (“Disable automatic network activity, including model catalog refreshes”), `PI_SKIP_VERSION_CHECK` (“Disable the `pi.dev` latest-version request”), `PI_SELF_UPDATE_COMMAND`, `PI_TELEMETRY` (`1`/`true`/`yes` or `0`/`false`/`no`), `PI_CODEMODE_MAX_WORKERS` (default 2, “Each worker's script may use up to 256 MB”), `PI_CACHE_RETENTION` (`long` for extended prompt caching), `PI_SHARE_VIEWER_URL`, `PI_RADIUS_GATEWAY`, plus display overrides (`PI_CLEAR_ON_SHRINK`, `PI_HARDWARE_CURSOR`, `PI_CLIPBOARD_OSC52`, `PI_HYPERLINKS`, `PI_PROGRAM_STATUS`, `PI_IMAGE_PROTOCOL`, `PI_TRUE_COLOR`, `PI_TUI_ESC_TIMEOUT` — “defaults to `100` over SSH and `10` otherwise”) and `VISUAL`/`EDITOR`/`HTTP_PROXY`/`HTTPS_PROXY`.

## Official sources

- [Configuration](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/configuration.md) · [Settings Reference](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/settings.md) · [Environment Variables](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/environment-variables.md) · [Keybindings](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/keybindings.md) · [Security (project trust)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md)
