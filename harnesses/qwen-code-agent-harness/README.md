# Qwen Code Agent Harness

**Verified:** 2026-11-16 (Qwen Code 0.27.4 npm latest; docs site `qwenlm.github.io/qwen-code-docs` main)

Qwen Code (repo `QwenLM/qwen-code`, npm `@qwen-code/qwen-code`, Node.js ≥ 22) is “The open-source AI coding agent for your terminal, editor, desktop, browser, and chat” — a fork of Gemini CLI grown into a multi-provider harness with a daemon/Web Shell, messaging channels, subagents, skills, and SDKs. It is **not** a Qwen model — models are configured providers, not the harness; “Qwen Code 1.0 Preview” is a separate preview line, and the main-line release documented here is 0.27.x. Registry alias: roster `qwen-code` under `qwts-qwen-agent`, normalized to profile alias `qwen`.

This set documents the terminal harness (`qwen` CLI, TUI, headless) plus its daemon, ACP, channel, and SDK surfaces. Credentials and account-specific tokens are excluded throughout.

**Verdict legend:** ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## In this folder

| File | Covers |
|---|---|
| [installation-auth-and-models.md](installation-auth-and-models.md) | Installers, the five `/auth` options, env keys, `modelProviders` config, `/model` + `/effort`, models.dev catalog, model slots |
| [configuration-and-settings.md](configuration-and-settings.md) | Seven configuration layers and four settings files, `QWEN_HOME`, `.env` loading/exclusions, settings sections, QWEN.md context files, rules, `.qwenignore`, CLI arguments |
| [sessions-checkpointing-and-memory.md](sessions-checkpointing-and-memory.md) | Session storage/resume/lock fencing, checkpointing (shadow Git), QWEN.md, auto-memory, Mem0, cross-session messaging |
| [tools-approvals-and-sandbox.md](tools-approvals-and-sandbox.md) | Built-in tools, the five approval modes and the Auto classifier, permission rules and bypass prevention, folder trust, Linux execution sandbox + Seatbelt/Docker/Podman, safe/bare mode |
| [subagents-skills-and-teams.md](subagents-skills-and-teams.md) | Named/fork/external-executor subagents, Agent Team, Arena, Agent Board, skills (+ `/learn`), custom commands, Goals |
| [extensions-hooks-and-mcp.md](extensions-hooks-and-mcp.md) | Extension sources (Gemini/Claude/Qoder/Agent Plugins v1), Mods, 24 hook events and the approval/trust policy, MCP transports/OAuth/filters |
| [automation-headless-serve-and-channels.md](automation-headless-serve-and-channels.md) | Headless formats/exit codes/budgets, Dual Output, `qwen serve` + Web Shell, scheduled tasks and channels, SDKs, GitHub Action, IDE/Computer Use/LSP surfaces |
| [corrections.md](corrections.md) | Claim-by-claim audit log (53 rows: 33 ❌ / 20 ⚠️) |

## Top corrections

The most consequential misconceptions this audit fixed (full log in [corrections.md](corrections.md)):

1. **“Qwen Code is a Qwen model / only runs Qwen models.”** ❌ It is an agent harness; “Supports OpenAI, Anthropic, Gemini, and Qwen APIs. Any third-party provider or local model (Ollama / vLLM).”
2. **“API keys can be stored in `settings.json`.”** ❌ “Credentials are never persisted in settings; the runtime reads them from `process.env[envKey]`.”
3. **“Folder trust is on by default.”** ❌ “The Trusted Folders feature is **disabled by default**. While it is disabled, every folder counts as trusted” (project MCP/hooks still ask; skills' frontmatter hooks do not).
4. **“Checkpointing is on by default.”** ❌ “The Checkpointing feature is disabled by default.”
5. **“Hooks are opt-in.”** ❌ “Hooks are enabled by default”; the disable switch is top-level `disableAllHooks`.
6. **“Cron is experimental and off by default.”** ⚠️ Inverted — `experimental.cron` defaults to `true`; terminal cron tasks are session-scoped and “gone when you exit.”
7. **“`permissions.deny` removes MCP tools.”** ⚠️ “MCP tools are exempt” from deny-based registry removal; deny still blocks them at runtime.
8. **“Shell commands bypass file rules.”** ❌ “Permission rules for `Read`, `Edit`, and `WebFetch` are also enforced when the agent runs equivalent shell commands.”
9. **“The Auto-mode classifier replaces permission rules and fails open.”** ❌ “Hard rules remain in force” before the classifier, over-broad allows are stripped in Auto Mode, and “when the classifier API is unreachable, the action is blocked.”
10. **“`qwen serve` loopback needs a token.”** ❌ “Loopback is trusted by default … every local process that can reach it receives full API authority, including code execution as the daemon user.”
11. **“`qwen auth coding-plan` configures the Coding Plan.”** ❌ Removed command — configure “with environment variables or `settings.json`.”
12. **“Subagents run in the foreground by default.”** ❌ “Top-level regular subagents run in the background by default.”

## Quick reference

| Area | Fact |
|---|---|
| Install | standalone installer (recommended) / `npm install -g @qwen-code/qwen-code@latest` / Homebrew; Node ≥ 22 |
| Auth (5) | Qwen OAuth subscription · QwenCloud (Token Plan `sk-sp-` / Standard `sk-`) · Alibaba ModelStudio (Coding/Token/Standard) · built-in third-party providers · custom OpenAI/Anthropic/Gemini-compatible |
| Config precedence | defaults < system-defaults.json < user `~/.qwen/settings.json` < project `.qwen/settings.json` < system settings.json < env vars < CLI args |
| Context files | `~/.qwen/QWEN.md` + ancestor/root `QWEN.md` + `.qwen/QWEN.local.md`; `AGENTS.md` read too; `@path` imports; `/memory`, `/init` |
| Rules | `.qwen/rules/`; `paths:`-gated conditional rules injected once; extension rules must be conditional |
| Approval modes | `plan`, `default` (Ask Permissions), `auto-edit`, `auto` (2-stage LLM classifier), `yolo`; `--allowed-tools` bypass list |
| Permission rules | `deny` > `ask` > `allow`, first match wins; `Read`/`Edit` meta-categories; path prefixes `//`, `~/`, `/`, `./`; shell bypass prevention |
| Sandbox | operator-only Linux `tools.executionSandbox` (bwrap/Landlock); whole-CLI Seatbelt profiles & Docker/Podman via `QWEN_SANDBOX` |
| Sessions | JSONL under `~/.qwen/projects/<sanitized-cwd>/chats`; `--continue`/`--resume`; writer-lease fencing; `/compress`, `/rewind`, `/branch`, `/export` |
| Checkpointing | off by default; shadow Git in `~/.qwen/history/<project_hash>`; `/restore` |
| Memory | auto-memory on by default; `/remember`, `/dream`; team memory opt-in; Mem0 via `memory.mem0` |
| Subagents | `task` tool; background by default; `fork` (fork_turns/fork_tools/fork_profile); `claude-code`/`codex` external executors |
| Teams/extras | Agent Team (`experimental.agentTeam`, `/coordinate`), Arena (`/arena`), Agent Board (`qwen board`) — all experimental |
| Skills | model-invoked `SKILL.md` folders; `~/.qwen/skills` + `.agents/skills`; `/learn` generation; `/skill-name` forces |
| Hooks | on by default; 4 types (command/http/function/prompt); 24 events; regex matchers; project hooks need approval or folder trust |
| MCP | `mcpServers` (user/project/system), `qwen mcp add`, `/mcp`; `httpUrl` > `url` > `command`; progressive discovery (~16 ms); OAuth with localhost:7777 default callback |
| Headless | `-p`; `text`/`json`/`stream-json`; `--input-format stream-json`; `--json-schema`; exit 53 turns / 55 budget / 130 SIGINT; `QWEN_CODE_UNATTENDED_RETRY` |
| Daemon | `qwen serve` on `127.0.0.1:4170` (loopback unauthenticated by default), Web Shell, SSE reconnect, multi-workspace single-operator |
| Scheduled | `/loop` + `CronCreate/List/Delete`; session-scoped, 50 max, 7-day recurring expiry, gates; channels use a separate persistent scheduler |
| Channels | `qwen channel start`; Telegram/WeChat/QQ/DingTalk/WeCom/Feishu/GitHub/GitLab/Email + webhook senders; `privatePolicy` default `pairing` |
| SDKs | `@qwen-code/sdk` (TS), `qwen-code-sdk` (Py ≥ 3.10), Java; daemon REST + OpenAPI |
| ACP | `--acp` (stable) for Zed and others; ACP bridge inside the daemon; `claude-agent-acp` executor |
| IDE | VS Code companion (Beta) with native diffing + workspace context; JetBrains page; Computer/Browser Use via `node-repl-mcp`; experimental LSP |

## Official sources

- [Docs site](https://qwenlm.github.io/qwen-code-docs/en/users/overview/) · [GitHub repo](https://github.com/QwenLM/qwen-code) · [npm](https://www.npmjs.com/package/@qwen-code/qwen-code) · [1.0 Preview branch](https://github.com/QwenLM/qwen-code/tree/v1.0.0-preview) · [Docs repo](https://github.com/QwenLM/qwen-code-docs)
