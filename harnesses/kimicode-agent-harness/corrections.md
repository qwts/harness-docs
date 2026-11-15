# Kimi Code corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| Kimi Code CLI is the legacy kimi-cli Python/uv package. | ❌ FALSE | The current CLI is `@moonshot-ai/kimi-code` on npm and requires Node.js ≥ 22.19; the Python/uv-era kimi-cli is the archived legacy product. |
| Providers can read credentials from shell environment variables like `export KIMI_API_KEY`. | ❌ FALSE | “It does not fall back to shell environment variables automatically: running export KIMI_API_KEY in the terminal does not give any provider its key.” Keys come from `api_key`/`api_key_env`, the provider `env` sub-table, or OAuth via /login. |
| A provider can set both `api_key` and `api_key_env`. | ❌ FALSE | They are “mutually exclusive alternatives — set exactly one”; supplying both fails config load. |
| Project-level `config.toml` files are supported. | ❌ FALSE | “The CLI currently reads a single user-level config file and has no project-level config file mechanism.” The project-side exceptions are permission rules (`.kimi-code/local.toml`, trusted workspaces) and project `mcp.json`. |
| K3.1 Pro can run with thinking disabled. | ❌ FALSE | “K3.1 Pro models only support thinking mode. Requests with thinking disabled are rejected.” |
| A mistyped managed model ID fails the request. | ❌ FALSE | “A mistyped model ID falls back to K2.8 Preview without thinking.” |
| `default_permission_mode = "yolo"` is the fully unattended mode. | ❌ FALSE | Yolo maps to Ask When Needed (regular calls auto-approved, sensitive operations still prompt); “Never Ask mode (formerly Auto), enabled with /auto, is the fully unattended mode.” |
| Permission rules are evaluated first-match-wins. | ❌ FALSE | Precedence is fixed: “matching deny rules take precedence over matching ask rules, which take precedence over matching allow rules.” |
| `dangerous_command_guard = false` only disables dangerous-shell-command confirmations. | ❌ FALSE | It turns “off every built-in guard confirmation policy entirely: dangerous commands, access to sensitive files such as .env or id_rsa, and access to git control paths.” Only “The hard block that stops Read/Write/Edit from touching secret files stays in effect.” |
| Hook definitions can ship in project files. | ❌ FALSE | “All hook rules are written in the [[hooks]] array in ~/.kimi-code/config.toml” (plus plugins) — “there is no project-level hook file, so content checked out on a remote target can never inject hook definitions into your session.” |
| A failing hook blocks the agent run. | ❌ FALSE | Hooks fail open: “Exit code: 0 means allow, 2 means block, other non-zero values default to allow”; only PreToolUse, Stop, and UserPromptSubmit are blockable. |
| The agent can enter Plan mode on its own. | ❌ FALSE | “Only you can turn Plan mode on; the agent never enters it on its own, even when you ask it to in chat.” |
| Never Ask mode removes every confirmation. | ❌ FALSE | “Exiting Plan mode always requires your confirmation, in every permission mode, including Never Ask.” |
| MCP tools always load into the top-level tool list. | ❌ FALSE | Deferred loading (experimental) keeps them out of the list — “the model first sees a manifest of loadable tools, loads full definitions on demand through the built-in select_tools tool.” |
| Project MCP servers load regardless of trust. | ❌ FALSE | “When Kimi Code finds project-level MCP servers in an untrusted folder, it shows each server's transport and launch target in the workspace trust prompt,” and headless runs keep them “disabled there unless the workspace is already trusted.” |
| `KIMI_CODE_TRUST_WORKSPACE=1` permanently trusts the workspace. | ❌ FALSE | It is a per-process override that “does not write a persistent trust record.” |
| Plugins can be installed per-project. | ❌ FALSE | “Plugins are currently installed per-user and apply to all projects; project-level installation scope is not yet supported.” |
| Plugin system-prompt contributions are unlimited. | ❌ FALSE | Each field is “limited to 32 KB (UTF-8 bytes)” and “one prompt build injects at most 64 KB of instructions”; excess is skipped with a warning. |
| Tower workers can spawn their own subagents. | ❌ FALSE | “Tower workers do not have the Agent tool and cannot spawn subagents.” |
| Tower worktrees are a security sandbox. | ❌ FALSE | “Worktrees isolate writes, but they are not a security sandbox. Workers can read repository content and run tools with your permissions,” and “Mission scopes and merge checks are process constraints, not security boundaries.” |
| Multiple sessions can coordinate Tower on one workspace. | ❌ FALSE | “Each workspace can have only one tower coordinator,” and after takeover “A project cannot have multiple tower coordinators at the same time.” |
| Remote environment commands can fall back to local execution. | ❌ FALSE | “There is no silent fallback to your machine, so a remote command cannot accidentally run locally.” |
| Hooks execute on the remote machine in remote environments. | ❌ FALSE | “hooks always execute on the machine running Kimi Code.” |
| The remote executor is installed automatically. | ❌ FALSE | “Kimi Code does not install the executor automatically. If it is missing or incompatible, install or update to the build provided for the target platform.” |
| Project permission rules apply in remote environments. | ❌ FALSE | “Remote project rules are not supported,” and local project rules need a trusted workspace. |
| Bash execution time is unbounded. | ❌ FALSE | Foreground defaults to 60 s with a 5-minute maximum; background tasks default to 10 min; auto-backgrounding applies on timeout. |
| Interactive commands work through the Bash tool. | ❌ FALSE | “stdin is always closed — interactive commands receive EOF immediately.” |
| HTTP 429 responses are always retried. | ⚠️ PARTIAL | “Retries only apply to transient failures: connection errors, timeouts, HTTP 429 rate limits, and 5xx server errors. A 429 caused by an exhausted quota or insufficient account balance is not retried and fails immediately.” |
| Subagents always run on the main agent's model. | ❌ FALSE | They inherit it by default, but “Subagents only run on models listed in the pool” once `[secondary_model]` is configured; non-pool aliases fail the spawn. |
| `secondary_model.force` pins the subagent model. | ❌ FALSE | “The former force key has been removed: a lone default_model already pins every subagent to one model. A configuration that still sets force loads with a warning and the key is ignored.” |
| Skills can nest arbitrarily. | ❌ FALSE | Nesting is capped at three levels. |
| `--agent` and `--agent-file` can combine. | ❌ FALSE | They are mutually exclusive; `--agent-file` loads one file at the highest priority. |
| Kimi Code quota refreshes weekly. | ❌ FALSE | “New members: the weekly quota limit is removed — quota no longer refreshes every 7 days. The rolling 5-hour rate window still applies,” with a monthly total quota shared across Kimi membership benefits. |
| Extra Usage is included in the subscription. | ❌ FALSE | It is a prepaid RMB balance (¥25 minimum top-up, 10 top-ups/¥3,000 per day, ¥10,000 cap) used after the subscription quota is exhausted. |
| The Kimi Code membership API and Kimi Open Platform API are the same service. | ❌ FALSE | Kimi Code uses api.kimi.com/coding (membership), while the Open Platform is api.moonshot.cn/v1 / api.moonshot.ai/v1 pay-as-you-go — separate accounts and billing. |
| `--dangerous-bypass-auth` is fine on a trusted LAN. | ❌ FALSE | It “turns off all authentication — anyone who can reach the port can control your sessions, file system, and shell.” |
| Remote Control is available to free users. | ❌ FALSE | “Remote Control requires a paid membership and is not available to free users,” and “Only one Remote Control instance can run per machine.” |
| The Read tool returns whole files. | ❌ FALSE | “The default and the most a call can request are both 100,000 characters,” and a call also stops at roughly a quarter of that in estimated tokens. |
| Grep returns all matches. | ❌ FALSE | Results are capped (`head_limit` 250) and “Sensitive files such as .env files and private keys are automatically filtered out.” |
| `kimi -p` prompts for tool approval. | ❌ FALSE | “In -p mode, no human approval is requested — regular tool calls are handled under the auto permission policy, while static deny rules remain in effect.” |
| The VS Code extension is a standalone implementation. | ❌ FALSE | It wraps the CLI (`kimi.executablePath` points at the binary; missing CLI is a documented error path). |
| Session data lives inside the project directory. | ❌ FALSE | Sessions live under `$KIMI_CODE_HOME/sessions/<workDirKey>/<sessionId>/` with `state.json`, `agents/main/wire.jsonl`, plans, tasks, and cron dirs. |
| Import from Claude Code migrates OAuth and MCP authorizations. | ❌ FALSE | “OAuth login credentials and MCP service authorizations are not copied, so you will need to run /login again and re-authorize MCP servers after migrating.” |
| AgentSwarm launches all subagents simultaneously. | ❌ FALSE | Ramp-up is rate-limited — “5 subagents start immediately, then 1 more every 700 ms” — with a 128-subagent cap. |
| Background-task timeouts behave the same in print mode. | ⚠️ PARTIAL | Print mode has its own semantics: `print_background_mode` is `steer` (default) / `drain` / `exit`, with dedicated `print_wait_ceiling_s` and `print_max_turns` fields. |
| Google Vertex credentials must be reconfigured inside kimi. | ⚠️ PARTIAL | `GOOGLE_APPLICATION_CREDENTIALS` is the one system-environment exception, “read by the Google SDK directly via the standard ADC flow; the CLI does not participate” — but Vertex still requires `GOOGLE_CLOUD_PROJECT`/`GOOGLE_CLOUD_LOCATION` in the provider env sub-table. |
| The status-line command can run as long as it needs. | ❌ FALSE | It is “capped at 300ms, throttled to once per second.” |
| Retries loop forever by default. | ❌ FALSE | Retry budgets are finite (`max_attempts_per_step` 10, `compaction_max_attempts` 5); indefinite retry is opt-in via `KIMI_CODE_INFINITE_RETRY`. |
| Resume needs the `--agent` flag to restore the bound agent. | ❌ FALSE | “The agent is bound at session creation, and resuming restores the bound agent automatically, so no flag is needed (or allowed) on resume.” |
| Resume cannot change the saved permission or plan mode. | ❌ FALSE | “When resuming a session, you can override its saved permission or plan mode by adding --auto, --yolo, or --plan. For example, kimi --continue --auto resumes the latest session and switches it to Never Ask mode.” The overrides page’s stricter exclusion line — “In non-prompt mode, --yolo and --plan cannot be combined with --continue or --session” — contradicts the kimi command reference and appears stale (doc-vs-doc; the reference’s fuller conflict list wins). |
| Filesystem watching runs by default. | ⚠️ PARTIAL | Doc-vs-doc inconsistency: the whats-new log states “Filesystem watching for config and workspace files no longer runs by default; to hot-reload AGENTS.md, skills, or MCP config, set [watch] enabled = true or KIMI_CODE_WATCH=1,” while the env-var reference still lists “`[watch]` enabled (default true).” |
