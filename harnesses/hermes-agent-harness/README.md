# Hermes Agent harness

**Verified:** 2026-11-15 against the official Hermes Agent documentation at hermes-agent.nousresearch.com/docs and the NousResearch/hermes-agent repository  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** `hermes` CLI/TUI/Desktop, `~/.hermes/` config and profiles, provider registry and model slots, approvals and security layers, skills/memory/context files, tools/MCP/hooks/cron, messaging gateway, Bot Mode, ACP and API server  
**Provenance:** Official Nous Research sources unless explicitly marked otherwise

Hermes Agent is Nous Research’s terminal-native autonomous agent — the overview positions it as “not a coding copilot” but a “self-improving AI agent” for interactive work, headless one-shots, 20+ messaging platforms, and scheduled automation. Installed via `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash` (MIT-licensed, `github.com/NousResearch/hermes-agent`), it keeps everything under `~/.hermes/` (`$HERMES_HOME`) with `config.yaml`/`.env`/`auth.json`, routes to a dozen-plus providers through a universal registry (Nous Portal is the convenience path, not a requirement), runs a four-system hook architecture, a 60-second gateway cron daemon, bounded MEMORY.md/USER.md memory with a frozen-snapshot prompt contract, and Bot Mode profiles that message each other. The `hermes` registry identity normalizes to this single harness; the Hermes model family and similarly named software are separate products.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, providers, and models](configuration-providers-and-models.md) | `~/.hermes/` layout, precedence CLI > config.yaml > .env > defaults, managed scope and its limits, external secret sources, provider registry and Nous Portal, auxiliary model slots, model_aliases, fallback chains, env vars |
| [Security, permissions, and sessions](security-permissions-and-sessions.md) | Approval modes and YOLO boundaries, file write safety, gateway authorization order, injection scanning, container hardening and MCP env filtering, SQLite sessions, checkpoints/rollback, git worktrees |
| [Skills, memory, and context files](skills-memory-and-context.md) | SOUL/USER/MEMORY/AGENTS file model, one-project-context rule, progressive discovery, bounded memory and the `memory` tool, write-approval gates, Skills Hub and trust model, project skills, bundles, `/learn`, curator, `/journey`, personalities |
| [Tools, MCP, hooks, and automation](tools-mcp-and-automation.md) | Toolsets, terminal backends and container security, MCP config/OAuth/catalog/serve, four hook systems with consent and fail-open/closed contracts, cron lifecycle and delivery, delegation, goals/loops/heartbeats, `/bg`, kanban |
| [Surfaces: CLI, gateway, Bot Mode, and integrations](surfaces-cli-gateway-and-integrations.md) | Install and setup, gateway-client CLI detachment and keybindings, TUI/Desktop, Bot Mode profiles and group chats, messaging gateway platforms, headless modes, ACP, API server and peers, product separation |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Hermes Agent runs only Nous’s own Hermes models.”** The registry spans openrouter, anthropic, gemini, zai, deepseek, bedrock, azure-foundry, ollama-cloud and more, plus custom providers; Nous Portal is the convenience path.
- ❌ **“Hermes runs YOLO by default and has no approval system.”** Approvals are smart/manual/off; YOLO is opt-in, the hardline blocklist stays, and `approvals.deny` globs are “evaluated before YOLO.”
- ❌ **“Memory auto-compacts when full.”** “Memory does **not** auto-compact” — the tool errors and the agent must consolidate in the same batch; saves are also a frozen snapshot, invisible to the running session.
- ❌ **“Hermes stacks `.hermes.md`, `AGENTS.md`, `CLAUDE.md`, and `.cursorrules` together.”** “Only **one** project context type is loaded per session, first match wins”; `SOUL.md` loads from `HERMES_HOME` only.
- ❌ **“Gateway hooks prompt for consent.”** They are “trusted-by-placement” — dropping `HOOK.yaml` + `handler.py` into `~/.hermes/hooks/` is the opt-in, with no enable list or prompt (shell hooks, by contrast, prompt once per pair).
- ❌ **“A failed shell hook aborts the tool call.”** Shell hooks fail open by default; `fail_closed: true` (pre_tool_call only) or exit code 2 blocks.
- ❌ **“The Docker backend starts a fresh container per command.”** One long-lived container per process; “behaves like a persistent sandbox VM, not a fresh container per command.”
- ❌ **“A scheduled agent can schedule more cron jobs by default.”** “Hermes disables cron management tools inside cron executions to prevent runaway scheduling loops”; opt-in via `cron.allow_agent_scheduling`.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Home | `~/.hermes/` (`$HERMES_HOME`); profiles in `~/.hermes/profiles/<name>/`; sessions in SQLite `state.db` (WAL, FTS5) |
| Precedence | CLI args > `config.yaml` > `.env` > defaults; managed-scope pins beat user values **and** the shell environment |
| Managed scope | `/etc/hermes/{config.yaml,.env}` (0755/0644, `HERMES_MANAGED_DIR` relocatable); per-key pinning; “filesystem permissions only” |
| Secrets | `.env` bootstrap; Bitwarden/1Password/command sources; “Your .env / shell wins by default”; child processes sanitized via `terminal.env_passthrough` |
| Providers | openrouter, nous, openai-codex, anthropic, gemini, qwen-oauth, zai, kimi-coding, minimax-oauth, deepseek, xai-oauth, ollama-cloud, bedrock, azure-foundry, custom |
| Portal | `hermes setup --portal`; Tool Gateway (web search, image gen, TTS, browser) is subscriber-only |
| Aux slots | vision, title_generation, compression, approval, mcp, tts_audio_tags, voice_chat, triage_specifier, kanban_decomposer, profile_describer, curator, delegation, background_review, review, group_router |
| Memory | MEMORY.md 2,200 chars / USER.md 1,375; `add`/`patch`/`remove` (no read, no replace); frozen snapshot; `write_approval` staging |
| Context files | One per session: `.hermes.md` → `AGENTS.override.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules`; SOUL.md independent from HERMES_HOME; 70/20/10 truncation |
| AGENTS chain | Git-root → cwd merged, deeper wins; progressive subdirectory discovery (≤5 ancestors, 32k hint cap, once per session) |
| Skills | `~/.hermes/skills/`; agentskills.io standard; progressive disclosure (index ~3k tokens); `/skill-name` slash commands (stack ≤5); bundles win slug collisions |
| Skill trust | Hub scan (dangerous verdicts unforceable); project skills need `hermes skills trust`; precedence project → local → create_dir → external |
| Approvals | smart/manual/off; `approvals.timeout` 300 s; destructive-slash confirm modal; `/approve`/`/deny` on messaging |
| YOLO | `--yolo` / `/yolo` / `HERMES_YOLO_MODE`; hardline blocklist floor; deny globs evaluated first |
| Hooks | Gateway (`HOOK.yaml`, trusted-by-placement), plugin (`ctx.register_hook`), shell (`hooks:` block, consented, exit-2 block, fail-open default), outbound webhooks (signed) |
| MCP | `mcp_servers` in config.yaml; `mcp_<server>_<tool>` naming; OAuth incl. device flow; lazy start; `/reload-mcp`; `hermes mcp serve` as a server |
| Terminal | local/docker/ssh/modal/daytona/vercel_sandbox (+singularity/sprites plugins); hardened containers; persistent docker VM |
| Cron | Gateway tick 60 s; jobs.json; pin → `cron.model` → main; no-agent scripts; `wakeAgent` gate; `context_from`/`continuity`; `[SILENT]`/`[CRON_FAILURE]` |
| Automation | `/goal` (judge, 20-turn budget), `/heartbeat` (≥60 s), `/loop`, `/bg`, delegate_task, kanban board |
| CLI | Gateway client over WebSocket; frozen session flags; `Esc Esc` backtrack; `!` shell mode; `/busy` modes; exit 3 on queued-owner loss |
| Bot Mode | Desktop built-in, on by default; Bot = profile; canonical Bot Chat (`/new` → `/compact`); routines = `[bot:<name>]` cron; `message_agent`; group chats 2–6 bots; `max_hops` 4 |
| Gateway | 20+ platforms; `/sethome`, `/pause`, `/platform`, `/restart`; authorization order platform-allow-all → DM-pairing → platform-allowlist → global-allowlist → global-allow-all → default-deny |
| Handoff | `/handoff <platform>` re-binds a CLI session to messaging with transcript replay |
| Headless | `hermes chat -q`; `--query-file` (verbatim); `-z` one-shot; detached-result runtimes fall back to synchronous |
| ACP | `hermes acp` (needs `acp` extra); curated toolset; frozen schemas; approvals allow once/session/always/deny (fail closed on timeout) |
| Checkpoints | Opt-in since v2; shadow git under `~/.hermes/checkpoints/store/`; `/rollback` keeps hand-edits by default |
| Worktrees | `hermes -w` / `/worktree new` under `.worktrees/`; kept when dirty/unpushed/active; 6-hour sweep archives before removing |
| Install | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh \| bash`; PowerShell/Termux/Nix/Docker paths; `hermes setup --portal` |

## Official sources

- [Overview](https://hermes-agent.nousresearch.com/docs) · [Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration) · [Configuring Models](https://hermes-agent.nousresearch.com/docs/user-guide/configuring-models) · [Managed Scope](https://hermes-agent.nousresearch.com/docs/user-guide/managed-scope) · [Secrets](https://hermes-agent.nousresearch.com/docs/user-guide/secrets)
- [Security](https://hermes-agent.nousresearch.com/docs/user-guide/security) · [Sessions](https://hermes-agent.nousresearch.com/docs/user-guide/sessions) · [Checkpoints & Rollback](https://hermes-agent.nousresearch.com/docs/user-guide/checkpoints-and-rollback) · [Git Worktrees](https://hermes-agent.nousresearch.com/docs/user-guide/git-worktrees)
- [Skills System](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) · [Persistent Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory) · [Context Files](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files) · [Which File Does What?](https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what)
- [Tools & Toolsets](https://hermes-agent.nousresearch.com/docs/user-guide/features/tools) · [MCP](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp) · [Event Hooks](https://hermes-agent.nousresearch.com/docs/user-guide/features/hooks) · [Scheduled Tasks (Cron)](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron)
- [CLI Interface](https://hermes-agent.nousresearch.com/docs/user-guide/cli) · [Bot Mode](https://hermes-agent.nousresearch.com/docs/user-guide/bot-mode) · [ACP](https://hermes-agent.nousresearch.com/docs/user-guide/features/acp) · [Slash Commands Reference](https://hermes-agent.nousresearch.com/docs/reference/slash-commands) · [llms.txt index](https://hermes-agent.nousresearch.com/docs/llms.txt)
