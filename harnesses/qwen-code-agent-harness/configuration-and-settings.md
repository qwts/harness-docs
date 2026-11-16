# Configuration and settings

**Verified:** 2026-11-16 (Qwen Code 0.27.4 npm latest; docs site main)

## Configuration layers

“Configuration is applied in the following order of precedence (lower numbers are overridden by higher numbers)”:

| Level | Source |
|---|---|
| 1 | “Default values” |
| 2 | “System defaults file” |
| 3 | “User settings file” |
| 4 | “Project settings file” |
| 5 | “System settings file” (overrides all other settings files) |
| 6 | “Environment variables … potentially loaded from `.env` files” |
| 7 | “Command-line arguments” |

Four JSON settings files: **System defaults** (`/etc/qwen-code/system-defaults.json`, macOS `/Library/Application Support/QwenCode/system-defaults.json`, Windows `C:\ProgramData\qwen-code\system-defaults.json`; overridable via `QWEN_CODE_SYSTEM_DEFAULTS_PATH`), **User** `~/.qwen/settings.json`, **Project** `.qwen/settings.json` (“Project settings override user settings”), and **System** `/etc/qwen-code/settings.json` (same-path overrides via `QWEN_CODE_SYSTEM_SETTINGS_PATH`; “May be useful for system administrators”). “String values within your `settings.json` files can reference environment variables using either `$VAR_NAME` or `${VAR_NAME}` syntax.” The old settings format “will be migrated automatically,” with a documented consolidation policy for `disableAutoUpdate`/`disableUpdateNag`.

`QWEN_HOME` “Customizes the global configuration directory (default: `~/.qwen`) … When set, project-level `.qwen/` directories are unaffected.” `QWEN_RUNTIME_DIR` overrides the runtime output directory (session records, logs, todos) over `advanced.runtimeOutputDir`.

## `.env` loading and exclusions

Qwen Code “can automatically load environment variables from `.env` files.” Precedence: “`<QWEN_HOME>/.env` (or `~/.qwen/.env` when `QWEN_HOME` is unset) is loaded before `~/.env`, and existing environment values are not overwritten.” Two exclusion rules:

- “Some environment variables (like `DEBUG` and `DEBUG_MODE`) are automatically excluded from project `.env` files by default” — configurable via `advanced.excludedEnvVars` (default `["DEBUG","DEBUG_MODE"]`); “Variables from `.qwen/.env` files are never excluded by this default list.”
- “**Loader-affecting variables are always rejected:**” — `NODE_OPTIONS`, `npm_config_node_options`, npm config-file redirects (`npm_config_userconfig`, `npm_config_globalconfig`, `npm_config_script_shell`, `npm_config_prefix`), `NODE_PATH`, `OPENSSL_CONF`, `NODE_REPL_EXTERNAL_MODULE`, `npm_config_node_gyp`, `npm_config_init_module`, `LD_PRELOAD`, `LD_AUDIT`, `DYLD_INSERT_LIBRARIES`, `BASH_ENV`, `ZDOTDIR`, and `BASH_FUNC_*` — “are never loaded from `.env` files (any scope, including `.qwen/.env` and user-level files) or from the top-level `settings.json` `env` section” — and the CLI “prints a warning when it rejects such a key.”

## Settings sections (highlights)

- **`permissions`** — see the tools and approvals page; `allow`/`ask`/`deny` rule arrays.
- **`tools`** — `approvalMode` (`plan`/`default`/`auto-edit`/`auto`/`yolo`), `sandbox`, `sandboxImage`, `executionSandbox` (operator-only Linux policy), `codeModeOnly` (experimental Code Mode, default `false`), `eager`/`disabled`/`discoveryCommand`, `lspEnabled` (experimental).
- **`memory`** — `enableManagedAutoMemory: true`, `enableManagedAutoDream: true`, `enableAutoSkill: false` (with `autoSkillConfirm: true`), `enableTeamMemory: false` (git-tracked `.qwen/team-memory/`, “Writes to it are secret-scanned”), `enableTeamMemorySync: false`, `enableStructuredRecall: false` (flat `MEMORY.md` listing vs “hierarchical memory tree, focused subtree, `search_memory` tool”; override `QWEN_CODE_MEMORY_STRUCTURED_RECALL=0|1`), `agentTimeoutMinutes`/`agentMaxTurns` for background memory agents, and `memory.mem0` for external Mem0 (`envKey` defaults to `MEM0_API_KEY`).
- **`agents`** — `defaultBackground: true`, `builtin.exploreModel: inherit`, `modelGrades`/`allowedGrades`, `crossSessionMessaging: true`, `crossSessionInbound` (`accept`/`hold`/`refuse`).
- **`security`** — `folderTrust.enabled: false`, `auth.selectedType`/`enforcedType`/`useExternal`, deprecated `auth.apiKey`/`auth.baseUrl`, `allowedInsecureVoiceBaseUrls` (default `[]`).
- **`serve`** — `maxSessions` (default `32`, “Honored from User, System, and SystemDefaults scopes only”), `maxConcurrentSubSessionsPerCaller` (`16`), `maxConcurrentSubSessionsTotal` (`24`).
- **`advanced`** — `autoConfigureMemory`, `dnsResolutionOrder`, `excludedEnvVars`, `bugCommand`, `plansDirectory` (“If unset, plan files are stored in `~/.qwen/plans`”).
- **`experimental`** — `generatedUi: true`, `mods: false`, `sessionWorkflow: false`, `cron: true`, `todoStopGuard: false`, `sessionWriterLease: false`, `cronRecurringMaxAgeDays: 7`, `agentTeam: false`, `unifiedAgentLoop: false`, `eventBusAgentLoop: false`, `artifact: true`, `emitToolUseSummaries: true`.
- **`mcpServers`** — MCP connections (see the extensions and MCP page); precedence “`httpUrl`, then `url`, then `command`.”
- **`telemetry`** — `enabled`, informational `target` (`local`/`gcp`), `otlpEndpoint`, `otlpProtocol` (`grpc`/`http`), `logPrompts`, `userId`, `includeSensitiveSpanAttributes`, `outfile`; every one overridable by `QWEN_TELEMETRY_*` variables. Usage statistics: `privacy.usageStatisticsEnabled` / `QWEN_USAGE_STATISTICS_ENABLED` — “Defaults to enabled when neither is configured.”
- **`general`** — includes `checkpointing.enabled` and `chatRecording` (headless `/goal` “requires `general.chatRecording` to remain enabled (the default)”).

## Context files (instructional memory)

Context files “defaulting to `QWEN.md` but configurable via the `context.fileName` setting” provide hierarchical instructions:

1. The global context file at `~/.qwen/<configured-context-filename>` — “Provides default instructions for all your projects.”
2. Project root and ancestor files — “The CLI searches for the configured context file in the current working directory and then in each parent directory up to either the project root (identified by a `.git` folder) or your home directory.”

“Content from files lower in this list (more specific) typically overrides or supplements content from files higher up (more general). The exact concatenation order and final context can be inspected from the `/memory` dialog.” Imports use “the `@path/to/file.md` syntax”; “Relative paths resolve from the QWEN.md file itself.”

The memory feature page adds the per-project variants: `~/.qwen/QWEN.md` (all projects), `QWEN.md` in the project root (team), and `.qwen/QWEN.local.md` (“Only you, only in this project”; loads “**after** the shared project `QWEN.md`”; “**You must gitignore it yourself**”). “If your repository already has an `AGENTS.md` file for other AI tools, Qwen reads that too.” `/init` analyzes the codebase to “create a starter QWEN.md”; “If one already exists, it suggests additions instead of overwriting.”

## Rules

Rules are Markdown files in `.qwen/rules/`: “`~/.qwen/rules/` (or `$QWEN_HOME/rules/`)” load **always**, “`<project>/.qwen/rules/`” load “when the workspace is trusted”, and extension `rules/` directories contribute “always, conditional rules only”. “**With `paths:`** — a _conditional_ rule. It stays out of the prompt until a tool call reads or edits a file matching one of its globs, and is then injected once for the rest of the session.” / “**Without `paths:`** — a _baseline_ rule. It is part of the system prompt from the first request, exactly like a context file, and costs the same on every turn.” “its rules must be conditional: a rule with no `paths:` is skipped, with a startup warning naming it,” and they “are not gated on workspace trust.” Globs match “against the path **relative to the project root**”, match dotfiles, resolve symlinks, and inject “**once per session**”.

## `.qwenignore`

“.qwenignore” excludes paths from supporting tools, “although they will still be visible to other services (such as Git).” Qwen Code reads `.qwenignore` plus “the compatibility files `.agentignore` and `.aiignore`” by default; `context.fileFiltering.customIgnoreFiles` adds others. “To apply the changes, you must restart your Qwen Code session.”

## Command-line arguments (selected)

`--model`/`-m`, `--prompt`/`-p`, `--prompt-interactive`/`-i`, `--system-prompt`, `--append-system-prompt`, `--output-style`, `--output-format` (`text`/`json`/`stream-json`), `--input-format`, `--include-partial-messages`, `--sandbox`/`-s`, `--sandbox-image`, `--debug`/`-d`, `--yolo`, `--approval-mode` (`plan`/`default`/`auto-edit`/`auto`/`yolo`), `--allowed-tools`, `--disabled-slash-commands`, telemetry flags, `--acp` (“Stable. Replaces the deprecated `--experimental-acp` flag”), `--experimental-lsp`, `--extensions`/`-e` (“Use the special term `qwen -e none` to disable all extensions”), `--list-extensions`/`-l`, `--proxy`, `--include-directories`, `--screen-reader`, `--version`, `--openai-logging`, `--openai-logging-dir`.

## Official sources

- [Settings](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/) · [Memory](https://qwenlm.github.io/qwen-code-docs/en/users/features/memory/) · [Rules](https://qwenlm.github.io/qwen-code-docs/en/users/features/rules/) · [Trusted Folders](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/trusted-folders/) · [Qwen Ignore](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/qwen-ignore/) · [Themes](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/themes/)
