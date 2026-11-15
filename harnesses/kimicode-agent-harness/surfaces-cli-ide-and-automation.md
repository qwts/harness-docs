# Surfaces: CLI, IDEs, Desktop, and automation

**Verified:** 2026-11-15

## The `kimi` command

`kimi` is the single binary (npm `@moonshot-ai/kimi-code`, Node ≥ 22.19; install script `curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash`). Interactive flags: `-S/--session [id]`, `-c/--continue`, `-m <alias>`, `-y/--yolo` (Ask When Needed; `--yes`/`--auto-approve` are aliases), `--auto` (Never Ask), `--plan`, `--skills-dir` (repeatable, replaces discovery), `--agent`, `--agent-file` (single file, highest priority, mutually exclusive with `--agent`), `--add-dir`, `--environment` / `--environment-cwd` (creation-only). Non-interactive is `-p/--prompt` with `--output-format <text|stream-json>` — “In -p mode, no human approval is requested — regular tool calls are handled under the auto permission policy, while static deny rules remain in effect” — and thinking is excluded from the JSONL stream. Conflicting flags fail at startup (the reference's Flag Conflict Rules: `--continue` vs `--session`, `--yolo` vs `--auto`, `--prompt` with `--yolo`/`--auto`/`--plan`, `--environment` with resume flags). “When resuming a session, you can override its saved permission or plan mode by adding --auto, --yolo, or --plan. For example, kimi --continue --auto resumes the latest session and switches it to Never Ask mode.” “Nobody can review a plan in non-interactive mode, so default_plan_mode is not applied to new sessions,” and “A resumed session that is already in Plan mode still runs and stays in Plan mode, but its plan cannot be approved in non-interactive mode.” (The overrides page still carries a stricter, stale exclusion line that contradicts the reference — logged in corrections.)

Subcommands: `login` (OAuth RFC 8628 device flow), `acp`, `web`, `app` (desktop launcher), `install-desktop`, `doctor`, `export`, `migrate`, `import-sessions`, `upgrade`, `vis`, `provider` (`add` / `remove` / `list`; `catalog list` + `catalog add` from models.dev api.json, `KIMI_REGISTRY_API_KEY` for private registries), and `rc` (Remote Control). The legacy `kimi server` tree is deprecated — only `server kill` remains for pre-0.28.0 processes.

## Web UI and Remote Control

`kimi web` serves a browser UI (default port 58627; `--port`, `--host`, `--allowed-host`); the token lives in `~/.kimi-code/server.token` and `kimi web rotate-token` invalidates old banner URLs. The blunt warning stands: “dangerous-bypass-auth turns off all authentication — anyone who can reach the port can control your sessions, file system, and shell.”

Remote Control pairs the machine with the Kimi mobile/web app: run “kimi rc in a terminal — it generates a link that can remotely control this machine.” Eligibility: “Remote Control requires a paid membership and is not available to free users,” and “Only one Remote Control instance can run per machine.” It “depends on a persistent connection between your machine and the Kimi service; remote sessions are unavailable after shutdown, sleep, or network loss,” and restart is by re-running “kimi rc again — there is no dedicated restart command.”

## Remote environments

`[environments]` declares SSH, Docker, or custom-launcher targets; `kimi --environment <name>` runs there, with the executor expected at `~/.kimi-code/bin/kimi` (`remoteBin` relocates it) — “Kimi Code does not install the executor automatically. If it is missing or incompatible, install or update to the build provided for the target platform.” “There is no silent fallback to your machine, so a remote command cannot accidentally run locally.” Still local even in remote mode: WebSearch/FetchURL, hooks, and stdio MCP servers without `environment_id`. Remote AGENTS.md/skills do not hot-reload; Tower is unavailable remotely; Windows targets support no Docker transport and need OpenSSH ≥ 8.4 for password prompts; directories over 50k entries are not enumerated.

## IDEs and ACP

The VS Code extension wraps the CLI: settings include `kimi.yoloMode` (auto-approve all tool calls), `kimi.autosave`, `kimi.editorContext` (never / onConversationStart / onFileChange), `kimi.useCtrlEnterToSend`, `kimi.enableNewConversationShortcut`, `kimi.alwaysExpandThinking`, `kimi.executablePath`, and `kimi.environmentVariables`. Context via `@src/app.ts:10-20` line ranges and Alt+K insert; media limits (paste ≤5 MB → compressed to ~2 MB, picker ≤10 MB images / 20 MB videos, 9 files / 80 MB per message); queued messages support ⚡ steer. The File Changes panel's “baseline state is captured at the first modification in the session; rollback will restore to this baseline version.” The action menu offers Working Directory / MCP Servers / General Config / Show Logs / Reset Kimi / Sign out, plus “Kimi Code: Run CLI” in the integrated terminal. Requests “time out after 30 seconds with no response.”

ACP (`kimi acp`) exposes the agent to editors — Zed, JetBrains AI chat, and Paseo are documented. Desktop install is `kimi install-desktop` or `kimi app`.

## Desktop app

The Desktop app (macOS 13+, Windows 10+, x86_64 Linux) wraps the same engine; data is shared under `~/.kimi-code/` plus app-shell state in `~/Library/Application Support/kimi-code-app/` (Windows: `%APPDATA%`). Task modes map to the CLI concepts — Normal, Plan, Goal, Swarm, Tower — with approval cards, a Changes panel, File Preview, a built-in terminal (Ctrl+`), and a built-in browser the agent can drive (take over, Annotate element). Side chat is read-only and not saved; forks can start from any turn; Settings span General / Account / Agent & Sessions / Model Providers / Remote Control / Plugins / Hotkeys / Browser / About / Lab / Archived Sessions; diagnostics export covers 30 days.

## Automation surfaces

Background tasks: foreground Bash defaults to 60 s (max 5 min), background tasks default 10 min (unbounded in print mode per `[task].print_background_mode`), auto-backgrounding on timeout, “A two-phase termination strategy (SIGTERM → 5-second grace period → SIGKILL),” and “stdin is always closed — interactive commands receive EOF immediately.” Cron: `CronCreate` takes a 5-field cron in local time (8 KB prompt cap, jitter, 50 active jobs max, 7-day stale auto-delete), with `CronList`/`CronDelete`; blocked in Plan mode; disabled by `KIMI_DISABLE_CRON`. Goal mode and Tower provide higher-level automation (see the permissions and agents pages). `kimi vis` renders a session timeline view.

## Official sources

- [Getting started](https://www.kimi.com/code/docs/en/kimi-code-cli/getting-started.html) · [kimi command](https://www.kimi.com/code/docs/en/kimi-code-cli/usage/kimi-command.html) · [Web](https://www.kimi.com/code/docs/en/kimi-code-cli/web.html) · [Remote Control](https://www.kimi.com/code/docs/en/kimi-code-cli/remote-control.html) · [Remote environments](https://www.kimi.com/code/docs/en/kimi-code-cli/remote-environment.html) · [IDEs](https://www.kimi.com/code/docs/en/kimi-code-cli/ides.html) · [ACP](https://www.kimi.com/code/docs/en/kimi-code-cli/kimi-acp.html) · [Desktop docs](https://www.kimi.com/code/docs/en/desktop/getting-started.html) · [VS Code docs](https://www.kimi.com/code/docs/en/vscode/getting-started.html)
