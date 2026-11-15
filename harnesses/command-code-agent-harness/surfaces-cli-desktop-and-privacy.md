# Surfaces, CLI, desktop, and privacy

**Verified:** 2026-11-15

## CLI: install, flags, headless

Install is `npm i -g command-code`; run `cmd` (`cmdc` on Windows, since `cmd` is the built-in shell — “use the full command-code, which works everywhere”). Updates install automatically unless `COMMANDCODE_SKIP_UPDATES` is set (`cmd update --check-only` for a manual check).

Key flags: `-p, --print [query]` (“Run in non-interactive mode, output response and exit”) with `--output-format text|json` (“newline-delimited AgentEvent frames plus one final result line”), `--max-turns` (headless run limit, default 100, “exits with code 8 if the cap is hit”), `--verbose`, `--skip-onboarding`; permission flags `--yolo`/`--dangerously-skip-permissions`, `--permission-mode <mode>`, `--accept-edits`, `--plan`, `-t/--trust`; model flags `-m/--model` (session-scoped; “Unknown ids are rejected”), `--list-models`, `--effort`; session flags `-r/--resume`, `-c/--continue`, `--session <path|id>`, `--fork-session`, `--no-session`, `-n/--name`; plus `-w/--worktree [name]` (“Accepts a name, a path, or a PR (#1234)”), `--config <key=value>` (repeatable headless `/config`), `--add-dir`, `--mod`, `--skill`, `--safe-mode`, `--no-skills`, `--local-only`. Subcommands: `info`, `status` (`--json`), `whoami`, `update`, `login`/`logout`, `mcp`, `skills`, `mods`, `taste`, `learn-taste`, `feedback`. CI auth is `COMMAND_CODE_API_KEY`.

Interactive mode: `/` opens the command menu (60+ built-ins), `!` runs bash with output joining the session, `@` is file-path autocomplete that also pulls in AGENTS.md files up the tree; the five daily keys are shift+tab (permission cycle), esc (stop), esc-esc (rewind), ctrl+o (tool output), ctrl+g (edit prompt in `$EDITOR`); ctrl+b backgrounds the running agent; alt+p opens the quick model switcher. Images attach via ctrl+v/alt+v (downscaled to 1200 px max, >32 MB skipped); text-only models can use “the VISION tool, one side-call to a cheap vision-capable model,” opt-in via `/config` → Image vision.

## Desktop app

Desktop (macOS/Linux/Windows, “currently in beta”) bundles “chat, agent activity, files, Git changes, a terminal, local browser previews, and plans in one workspace” with “the agent runtime … built in, so you do not need to install the Command Code CLI.” Its permission modes are Ask for approval, Auto-accept edits, Plan, and Full access — and “Shift+Tab skips Full access” there, unlike the CLI. Desktop lacks CLI features: headless mode, `/share`, scheduled tasks, start-up flags, subcommands, the `dont-ask` mode, and several slash commands (`/add-dir`, `/agents`, `/login`, `/memory`, `/todos`); `/config`, `/theme`, and `/mcp` map to Settings pages, and MCP settings are project-scoped. Note on shipping: “A commit includes all current project changes, not only changes made by the agent.”

## Other surfaces

The docs index lists IDE extensions and IDE integration, ACP (“Agent Client Protocol”), Vision, Cloud (“Cloud sessions with sandboxes”, Beta), Studio (“Taste profiles. API keys. Teams.”), remote control, Artifacts, and scheduled work (`/loop`, `/goal`, `cron/jobs.json`) as first-class pages; this doc set does not deep-dive those beyond the CLI/desktop surface facts verified above.

## Privacy, telemetry, and local-only mode

The security page’s data table: source code “Your machine only”; conversation history “Local only” in `~/.commandcode/projects/`; auth tokens “Local only” in `auth.json`; AGENTS.md “Local only”; taste profile “Local + optional cloud sync.” “Command Code does not make any network requests for telemetry or tracking without your knowledge”; outbound connections are limited to provider API calls, `cmd login` OAuth, optional taste sync, and configured MCP servers. `DO_NOT_TRACK=1` stops telemetry exporters; `CMD_ZDR=1` “enforces zero data retention on every request.”

Local-only mode (`cmd --local-only`, `CMD_LOCAL_ONLY=1`, or `"localOnly": true`) guarantees “No telemetry, anywhere,” disables billing reads, and turns off the Command Code catalog, `/share`, `/usage`, server-proxied `web_search`/`web_fetch`, taste push/pull, and backend agent generation — “Your BYOK providers, and everything local keep working exactly as before.” File access is bounded to the project directory, `--add-dir`/`/add-dir` additions, and `~/.commandcode/`: “It does not read files outside these boundaries.” Enterprise options advertised: self-hosted deployment, “Your code never leaves your infrastructure,” and custom security policies.

The harness-engineering docs describe the efficiency layer behind the product’s open-model positioning — tool-call repairs (“Free on every plan”), the read and shell tools ranked “#1 of 10 harnesses” in the vendor’s own benchmarks, and cache work — presented as vendor claims, not independent measurements.

## Official sources

- [CLI Reference](https://commandcode.ai/docs/reference/cli)
- [Interactive Mode](https://commandcode.ai/docs/interactive-mode)
- [Desktop App](https://commandcode.ai/docs/desktop)
- [Windows](https://commandcode.ai/docs/windows)
- [Security & Privacy](https://commandcode.ai/docs/resources/security)
- [Zero Data Retention](https://commandcode.ai/docs/resources/zdr)
- [Harness Engineering](https://commandcode.ai/docs/harness-engineering)
- [Docs index](https://commandcode.ai/docs)
