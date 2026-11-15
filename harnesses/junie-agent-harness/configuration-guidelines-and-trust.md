# Configuration, guidelines, and trust

**Verified:** 2026-11-15

## Junie home

Everything lives under `~/.junie/` by default; `$JUNIE_HOME` relocates it (`-` in the env table — no CLI flag). Documented contents include `config.json`, `settings.json`, `allowlist.json`, `sandbox.json`, `trust/`, `mcp/mcp.json`, `skills/`, `agents/`, `commands/`, `models/`, `extensions/`, `themes/`, `teleport/`, `logs/junie.log`, and `sessions/` (the GitLab CI wrapper archives `~/.junie/sessions`, `~/.junie/logs`). The cache directory can be moved independently with `--cache-dir`.

## Two configuration channels

**config.json** is the documented configuration file, loaded from:

- User scope: `~/.junie/config.json` — “personal defaults on your machine”
- Project scope: `<project-root>/.junie/config.json` — “settings that should be shared with the whole team”

Merge precedence (highest first): **command-line flags → project config (when the CLI project is trusted) → user config**. Relative paths inside a config file resolve “relative to the folder that contains that configuration file.”

**settings.json** (`~/.junie/settings.json`) is “a separate channel and … not part of this config.json merge”: “There is no single global precedence across both channels: settings such as the model, provider, brave mode, and effort each resolve through their own consumer path.” Documented settings.json residents: the `/settings` UI values (theme, notifications, step limit, subagents mode, diff view, Show transcript), the subagents `SameModelOnly`/`Auto` policy (EAP), and the `cache*` family (see below).

Extra config files load with `--config-location` (repeatable, also `JUNIE_CONFIG_LOCATION`); explicit locations load “even when the CLI project is untrusted.” `--config-default-locations false` (env `JUNIE_CONFIG_DEFAULT_LOCATIONS`) disables both default locations.

Supported config.json fields: `model`, `effort`, `provider`, `brave`, `flags`, `mcp-locations` / `mcp-default-locations`, `skill-locations` / `skill-default-locations`, `command-locations` / `command-default-locations`, `agent-locations` / `agent-default-locations`, `model-locations` / `model-default-locations`, `auto-update`, `guidelines-location`, `byok` (per-provider API keys), `proxies`, `hooks`.

## Project trust

Interactive CLI sessions ask for a trust decision before loading project-scoped inputs from a project without a valid stored trust marker:

- **Keep untrusted** — “continue with isolated temporary project Junie storage”
- **Trust this project** — exact trust only
- **Trust all projects in `<parent>`** — ancestor trust, “using path-aware containment rather than string-prefix matching”

Junie “canonicalizes project and scope paths and resolves symbolic links before evaluating trust.” Trust markers live under `~/.junie/trust`; each holds “its kind and canonical path, but cannot grant trust unless its integrity code matches that key” — a project-trust authentication key stored in macOS Keychain, Windows Credential Manager, or the Linux Secret Service, with an owner-only file fallback “inside the same trust directory” when native secure storage is unavailable. “Neither the key nor trust markers are written to settings.json or the plaintext secure_credentials.json fallback.” Deleting an exact marker revokes that project; deleting a parent marker revokes inherited trust on the next CLI process. Keeping a project untrusted does not create a denial marker. If a marker cannot be written and verified, the choice applies to the current run only and Junie says so once.

An untrusted project “remains the workspace for ordinary file operations,” but Junie “does not implicitly load project configuration, MCP servers, hooks, extensions, models, plans, demos, custom agents or commands, skills, root or project Junie guidelines, project memory, or automatic migration/onboarding sources”; instead it uses “a writable temporary project Junie directory outside the repository” — MCP servers, skills, and commands added during the session land there “and are removed when the CLI process closes.” Global sources under Junie Home stay enabled, and “paths supplied explicitly through CLI options or environment variables, including `--config-location`, remain enabled because the user selected them deliberately.”

Non-interactive launches are trusted by design: “Non-interactive JSON, ACP, and Gateway launches are always trusted: they cannot ask you for a decision, so they load project configuration without a prompt, a flag, or an environment variable.”

## Environment variables

CLI flags outrank env vars: “For settings that exist as both environment variables and command-line flags, command-line flags take priority.” Full table in the docs; the load-bearing ones:

| Variable | Purpose |
|---|---|
| `JUNIE_API_KEY` | Junie API key auth (`--auth`) |
| `JUNIE_ANTHROPIC_API_KEY` / `JUNIE_OPENAI_API_KEY` / `JUNIE_GOOGLE_API_KEY` / `JUNIE_GROK_API_KEY` / `JUNIE_META_API_KEY` / `JUNIE_OPENROUTER_API_KEY` | BYOK provider keys |
| `JUNIE_LITELLM_URL` / `JUNIE_LITELLM_API_KEY` | LiteLLM proxy |
| `JUNIE_MODEL` / `JUNIE_LLM_PROVIDER` / `JUNIE_EFFORT` | Model selection; effort requires a model or agent |
| `JUNIE_TASK` / `JUNIE_PROMPT` / `JUNIE_SYSTEM_PROMPT` / `JUNIE_PROJECT` | Headless task, interactive initial prompt, process-local system prompt, project dir |
| `JUNIE_HOME` | Junie home override |
| `JUNIE_CONFIG_LOCATION` / `JUNIE_CONFIG_DEFAULT_LOCATIONS` | Extra config files, default-location toggle |
| `JUNIE_MCP_LOCATIONS` / `JUNIE_MCP_DEFAULT_LOCATIONS`, `JUNIE_SKILL_*`, `JUNIE_COMMAND_*`, `JUNIE_AGENT_*`, `JUNIE_MODEL_LOCATIONS` / `JUNIE_MODEL_DEFAULT_LOCATIONS` | Discovery locations per feature |
| `JUNIE_AGENT` | Custom agent as the session’s main agent |
| `JUNIE_EXTENSIONS_DEFAULT_LOCATION` | Extensions dir override |
| `JUNIE_SHARE_ANONYMOUS_STATISTICS` | Per-run telemetry override |
| `JUNIE_STOP_HOOK_BLOCK_CAP` | Stop-hook block cap override (0 disables) |
| `JUNIE_GUIDELINES_FILENAME` | Custom guidelines file (placed in `.junie/`) |
| `INGRAZZIO_URL` | Legacy implicit Ingrazzio proxy |

`JUNIE_SYSTEM_PROMPT` will “Add process-local user guidelines to the top-level issue agent (and to a custom agent launched with `--agent`) in every session”; it is “not persisted or passed to chat agents or subagents,” and an ACP client session prompt “is appended after this value on a new line.”

## Guidelines

“Junie CLI reads guidelines from the AGENTS.md file and adds this context to every task it works on.” Discovery order (first match wins):

1. `.junie/AGENTS.md` — “used exclusively; no other guidelines files are combined with it”
2. Root `AGENTS.md`, “combined with `.junie/playbook.md` and every `.junie/rules/*.md` file, if present” — concatenated in that order
3. `.junie/guidelines.md` or `.junie/guidelines/` folder — “Junie’s legacy format for guidelines (still supported)”, used only when none of the above exist

`--guidelines-filename` / `JUNIE_GUIDELINES_FILENAME` points at a custom file in `.junie/`; when set and present it is used exclusively. config.json’s `guidelines-location` field does the same declaratively.

**Global guidelines:** `~/.junie/AGENTS.md` (`%USERPROFILE%\.junie\AGENTS.md` on Windows) — if only one of global/project exists, it is used as-is; if both exist “Junie includes both and marks them clearly. Project-level guidelines always take precedence over global ones when they conflict”; identical content is deduplicated — “Junie automatically deduplicates and uses the content only once.”

**Import from other agents:** Junie “checks for any guidelines or memory files from other AI agents when it opens the project for the first time. If such files are detected, it will suggest importing the instructions into `.junie/AGENTS.md`.” The `/import` command handles this explicitly — “detects and suggests import of guidelines, agent skills, slash commands, or MCP server configurations from other coding agents like Claude Code, Codex, or Cursor.”

## Custom slash commands

Markdown files with YAML frontmatter (a `description` field) in `.junie/commands/` (project, committable) and `~/.junie/commands/` (user). The file name is the command name. Prompt bodies accept `$argumentName` placeholders — “A custom slash command will only be executed when all arguments from the command template are provided,” invoked as `/explain file=src/main.kt` with quoted or unquoted values and Tab completion for missing names. “By default, custom slash commands accept only the named arguments defined in the command template”; `allowPromptArgument: true` in the frontmatter exposes `$prompt` (or appends free text as “User Input: ...”). Manage via `/commands`.

## Cache and settings.json

`junie cache` reports disk usage (versions, update archives, logs); `junie cache clean [--dry-run] [--sessions]` prunes. “Saved sessions are your data, not cache. Junie CLI never deletes them unless you explicitly ask for it with `--sessions` or enable the cacheAutoPruneSessions setting.” Defaults: keep 3 versions, `cacheRecentVersionDays` 14 (5 nightly), update archives 7 days, logs 30 days, sessions 90 days (finished ones only, and “sessions that are open in any running Junie CLI process … are also kept, however old they are”). Automatic background pruning runs “at most once every 24 hours” in interactive/ACP sessions, “only for Junie CLI installed with the install script and with auto-updates enabled. It does not run in headless mode.” Cache settings live in `~/.junie/settings.json` and “are not read from the project config.json file”; invalid values are ignored with a warning.

## Themes

`/themes` switches among built-ins (`Auto`/`Dark`/`Light` — Auto follows the terminal background) and custom JSON themes in `~/.junie/themes/` (or `$JUNIE_HOME/themes/`), loaded only at startup. A theme file supports `extends` (“dark” or “light”), `defs` (named colors), and a `theme` token map; colors are `#RRGGBB` hex or palette references — “The short form (#RGB) and values with an alpha channel (#RRGGBBAA) are not supported.”

## Official sources

- [config.json](https://junie.jetbrains.com/docs/junie-cli-configuration.html) · [Guidelines and memory](https://junie.jetbrains.com/docs/guidelines-and-memory.html) · [Environment variables](https://junie.jetbrains.com/docs/environment-variables.html) · [Custom slash commands](https://junie.jetbrains.com/docs/custom-slash-commands.html) · [Cache management](https://junie.jetbrains.com/docs/junie-cli-cache.html) · [Color themes](https://junie.jetbrains.com/docs/junie-cli-themes.html) · [CLI reference](https://junie.jetbrains.com/docs/parameters.html)
