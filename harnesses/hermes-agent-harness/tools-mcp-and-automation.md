# Tools, MCP, hooks, and automation

**Verified:** 2026-11-15

## Toolsets

Built-in tools are grouped into toolsets enabled per platform: `web`, `search`, `terminal`, `file`, `browser`, `vision`, `image_gen`, `skills`, `tts`, `todo`, `memory`, `session_search`, `cronjob`, `code_execution`, `delegation`, `clarify`, `messaging`, `discord`, `discord_admin`, `debugging`, `safe` — plus `x_search` (off by default, xAI-gated) and dynamic `mcp-<server>` toolsets. `hermes tools` is the interactive per-platform editor; `agent.disabled_toolsets` is the hard switch; `platform_toolsets.<platform>` overrides per surface.

## Terminal backends

| Backend | Notes |
|---|---|
| `local` | Default; runs on your machine |
| `docker` | One persistent container per Hermes process (`docker exec` into it); “behaves like a persistent sandbox VM, not a fresh container per command” |
| `ssh` | “Recommended for security — agent can’t modify its own code” |
| `modal`, `daytona`, `vercel_sandbox` | Cloud execution (Vercel needs all three of `VERCEL_TOKEN`/`VERCEL_PROJECT_ID`/`VERCEL_TEAM_ID`) |
| `singularity`, `sprites` | Plugins (HPC containers; stateful Fly.io sandboxes) |

All container backends drop capabilities, set pids-limit 256, and use namespace isolation. Background work uses the `process` tool (`list`/`poll`/`wait`/`log`/`kill`/`write`); a trailing `&` command auto-backgrounds with an `auto_backgrounded` note; repeated waits on a quiet process flip on `notify_on_complete`; completed receipts persist 64 results for 7 days under `logs/process-results/`.

## MCP

Servers configure under `mcp_servers` in `config.yaml` — stdio (`command`/`args`/`env`) or HTTP (`url`/`headers`, `auth: oauth` with PKCE, pre-registered clients for no-DCR vendors, device-code and paste-back flows for headless hosts, mTLS via `client_cert`). Notable keys: `tools.include`/`tools.exclude` (fnmatch globs; `include` wins), `tools.prompts: false`/`tools.resources: false`, `lazy: true` (register from schema cache, connect on first call), `idle_timeout_seconds`/`max_lifetime_seconds` (recycle memory-heavy servers like Playwright transparently), `supports_parallel_tool_calls`, `identity_header`, `sampling` (on by default; rate-limited, capped), and `elicitation` (form-mode routed through the approval surface; URL-mode declined).

Tools register as `mcp_<server>_<tool>`; discovery runs 4 servers at a time per pass; `notifications/tools/list_changed` re-fetches without `/reload-mcp`; the gateway watches `config.yaml` and applies edits within about a minute. The curated catalog (`hermes mcp`) ships Nous-reviewed entries only — “There is no community submission tier” — and GitHub is deliberately absent (“its hosted MCP requires each client to bring its own OAuth app”).

Hermes can also **be** an MCP server: `hermes mcp serve` exposes 10 messaging-bridge tools (`conversations_list`, `messages_send`, `permissions_respond`, …) over stdio; sends require the gateway running.

## Hooks — four systems

| System | Registered via | Runs in | Consent |
|---|---|---|---|
| Gateway hooks | `HOOK.yaml` + `handler.py` under `~/.hermes/hooks/` | Gateway only | “trusted-by-placement” — placing the files is the opt-in |
| Plugin hooks | `ctx.register_hook()` | CLI + Gateway | Explicit (`plugins.enabled`) |
| Shell hooks | `hooks:` block in `config.yaml` | CLI + Gateway + Desktop/TUI/dashboard | First-use prompt per `(event, command)`, persisted allowlist; escapes: `--accept-hooks`, `HERMES_ACCEPT_HOOKS=1`, `hooks_auto_accept: true` |
| Outbound webhooks | `hooks.outbound:` list | CLI + Gateway | None (notify-only, HMAC-SHA256-signed, one retry) |

Key plugin/shell-hook semantics: `pre_tool_call` is directive — “any valid `block` wins over any `approve`”, `modify` shallow-merges args, a hung callback **fails closed**; `pre_llm_call` injects context into the user message (never the system prompt — “This preserves the prompt cache”) or blocks the turn before inference; `pre_verify` can continue the turn at the verify gate (capped by `agent.max_verify_nudges`, default 3); transforms (`transform_tool_result`, `transform_terminal_output`, `transform_llm_output` with `hold_stream`) rewrite content. Shell hooks speak JSON over stdin/stdout, accept Claude-Code-compatible shapes, block on exit code 2, and fail open unless `fail_closed: true` (pre_tool_call only; `timeout` default 60, capped at 300). Python plugins register first, so their decisions win ties.

## Cron

The gateway daemon ticks every 60 s over `~/.hermes/cron/jobs.json`; jobs run in fresh isolated sessions with the `cron` platform’s toolset (or `enabled_toolsets` per job). Model resolution: per-job pin → `cron.model` → main model at fire time; per-job reasoning pins override globals. Schedule formats: `in 30m`, `every 2h`, `every monday 9am`, cron expressions, ISO timestamps.

- **No-agent mode** (`--no-agent --script`): script stdout delivered verbatim, “no tokens, no model”; empty stdout a silent tick; scripts must resolve inside `$HERMES_HOME/scripts/`.
- **Gates and chains:** a `wakeAgent: false` stdout line skips the LLM for a tick; `context_from` prepends another job’s latest output; `continuity=true` dedupes against the job’s own last report.
- **Delivery:** `deliver:` targets (`origin`, `local`, `telegram:123456`, `discord`, `slack`, `all`, comma fan-out, `bot-chat[:profile]`); “the agent does not send messages itself.” `[SILENT]` suppresses delivery on success; `[CRON_FAILURE]` on the first line declares a failed run. Delivery failures book `last_status: delivery_failed`, distinct from run failures.
- **Ops:** `hermes cron pause/resume/run/cancel/remove/edit/list/status/doctor/incidents`, `hermes pause` (global stop), `cron.catch_up_missed`, `--grace-minutes`, failure incidents alert once per error signature with a cooldown, and `cron.allow_agent_scheduling: true` to let scheduled agents manage jobs (off by default).

## Delegation and multi-agent

`delegate_task` runs child agents (roles like `leaf`/`orchestrator`); pinned children never use the fallback chain. `/review` spawns “an independent, full-privilege reviewer subagent” whose review re-enters the session. `/bg` runs prompts in separate background sessions; `/btw` answers from a read-only transcript snapshot. **Kanban** provides a multi-profile, multi-project board with dispatcher/worker lanes and its own hook family. The Codex app-server runtime (`/codex-runtime`) hands eligible OpenAI/Codex turns to a native `codex app-server` subprocess.

## Session-scoped automation

- `/goal <text>` — a standing goal with an auxiliary judge model after each turn; auto-continues until done, paused, or `goals.max_turns` (default 20); real user messages preempt it.
- `/subgoal <text>` — appends criteria the judge must also satisfy.
- `/heartbeat every <interval> <prompt>` — re-enters the session when idle (min 60 s, missed ticks coalesce); in-process only.
- `/loop [interval] <prompt> [--times N] [--until <condition>]` — recurring re-fires with a judge-able stop condition.

All three are session-scoped; durable schedules belong in cron.

## Official sources

- [Tools & Toolsets](https://hermes-agent.nousresearch.com/docs/user-guide/features/tools) · [MCP](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp) · [Event Hooks](https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks) · [Scheduled Tasks (Cron)](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron) · [Subagent Delegation](https://hermes-agent.nousresearch.com/docs/user-guide/features/delegation) · [Persistent Goals](https://hermes-agent.nousresearch.com/docs/user-guide/features/goals) · [Session Heartbeats](https://hermes-agent.nousresearch.com/docs/user-guide/features/heartbeat) · [Recurring Loops](https://hermes-agent.nousresearch.com/docs/user-guide/features/loops)
