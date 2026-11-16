# Warp Agent Harness (Warp)

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main, changelog 2026.11.05 / app v0.2026.11.04.08.17; `warpdotdev/warp` AGPL-3.0; Warp Agent CLI quickstart/reference current)

Warp is “an **Agentic Development Environment**” — “Warp is an agentic development environment, born out of the terminal” (`warpdotdev/warp`, AGPL-3.0, docs index). Its built-in agent is the **Warp Agent**: “Warp's built-in coding agent. It works through multi-step tasks on its own, running commands in a real terminal and using the output to decide what to do next. You stay in the loop, approving actions and reviewing diffs before anything lands.” Surfaces: “**You reach the same agent three ways**” — the Warp app, “In any terminal, with the Warp Agent CLI” (the `warp` binary, “a standalone terminal program that runs the Warp Agent, the same agent that powers the Warp app” and “a native terminal multiplexer. It creates and maintains its own pseudoterminal (PTY)”), and cloud agents on the Automation Platform (formerly Oz). The legacy cloud-management CLI is the `oz` binary, “which previously shipped as `warp-cli`,” and is “a supported legacy interface.” Registry alias: roster `warp` under `qwts-warp-agent`. This reference documents the local Warp Agent (app + CLI) and the cloud surface at the boundary; Warp Factories internals, third-party CLI-agent pages, and enterprise administration are out of scope except where they bound the agent.

Version anchors: app ships weekly (“Warp ships weekly updates, typically on Thursdays”; latest changelog entry 2026.11.05, v0.2026.11.04.08.17); Agent Memory is a research preview; the Oz→Automation-Platform naming transition is explicitly ongoing (“Warp will publish guidance before any support change”). Credentials and account-specific secrets are excluded throughout.

**Verdict legend:** ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## In this folder

| File | Covers |
|---|---|
| [installation-and-surfaces.md](installation-and-surfaces.md) | The three agent surfaces, app install, `warp` CLI install and self-update, legacy `oz` CLI, browser device-code login, `WARP_API_KEY` for CI/headless, `/status`–`/usage`–`/team` |
| [configuration-context-and-skills.md](configuration-context-and-skills.md) | `settings.toml` paths and hot reload, Rules (`AGENTS.md`/`WARP.md`, precedence, all-caps), skills (`SKILL.md`, `$ARGUMENTS`, conflict resolution, bundled skills), Codebase Context, agent questions, notifications, no-hooks note |
| [permissions-and-approvals.md](permissions-and-approvals.md) | `agent_decides`/`always_ask`/`always_allow`, execution profiles in TOML, allowlist/denylist precedence, auto-approve and denylist bypass, approval cards, Full Terminal Use |
| [models-and-inference.md](models-and-inference.md) | Model picker and Auto models, provider `model_id`s, BYOK (device secure storage, credit fallback), custom routers (YAML), ChatGPT/SuperGrok subscriptions, usage levels |
| [sessions-planning-and-orchestration.md](sessions-planning-and-orchestration.md) | Terminal/Agent modes, CLI transcript and `/plan`, resume and `/fork`, orchestration approval gate, handoff (fork + snapshot), cloud agents, scheduled agents, Computer Use, Factories boundary, third-party CLI agents |
| [mcp-and-integrations.md](mcp-and-integrations.md) | App MCP (transports, file-based configs, third-party provider configs, auto-spawn rules), CLI MCP (global-only config), platform integrations and secrets, Platform API/SDK (no ACP) |
| [corrections.md](corrections.md) | Claim-by-claim audit log (31 rows: 25 ❌ / 6 ⚠️) |

## Top corrections

1. **“Warp is just an AI-powered terminal.”** ❌ Warp is “an **Agentic Development Environment**” — terminal, agent app + CLI, cloud Automation Platform, and Warp Factories.
2. **“You need the Warp app to use the agent.”** ❌ “The CLI doesn't require the Warp app to be installed” and works “in any terminal emulator, including over SSH.”
3. **“The Warp Agent CLI is the `warp-cli` command.”** ❌ `warp-cli` was the old name of the cloud CLI — the `oz` binary, “a supported legacy interface”; the `warp` binary is the new local agent CLI.
4. **“The agent's commands run in a sandbox.”** ❌ Locally it runs shell commands in a real terminal/PTY (“drives interactive programs like REPLs, database shells, and full-screen apps”); sandboxes live in Computer Use and cloud environments.
5. **“Auto-approve still enforces your command denylist.”** ❌ “With auto-approve on, the agent runs commands and applies file edits without review, including commands that match your own command denylist” (opt-out in settings; Admin-Panel denylists always apply).
6. **“`agent_decides` auto-applies file edits.”** ❌ “For **Apply code diffs**, `Agent decides` currently behaves the same as `Always ask` … Only `Always allow` skips the review prompt.”
7. **“BYOK keys are stored on Warp's servers.”** ❌ “Warp never stores provider keys on its servers” — device secure storage, CLI and app stored separately.
8. **“BYOK works for cloud runs.”** ❌ “Personal BYOK keys work for local Warp Agent runs, not cloud runs” (team-managed keys are the cloud path).
9. **“Auto models bill through your provider after BYOK.”** ❌ “Warp's built-in Auto models always use Warp-provided inference, even with BYOK configured.”
10. **“Project-scoped MCP servers start automatically.”** ❌ “Project-scoped servers from any provider require explicit approval,” are session-scoped, and never auto-spawn from cloned repos.
11. **“Cloud child agents can be steered from the CLI.”** ❌ “Cloud child sessions in the CLI only show status. They don't accept input.”
12. **“Computer Use works in your local session.”** ❌ “Computer Use is only available in Warp's sandboxed cloud environments, not in local interactive terminal sessions.”

## Quick reference

| Area | Fact |
|---|---|
| Product | “an **Agentic Development Environment**” (open-source, AGPL-3.0); terminal + Warp Agent + Automation Platform (ex-Oz) + Warp Factories |
| Agent | “Warp's built-in coding agent”; multi-step, real terminal, “You stay in the loop, approving actions and reviewing diffs before anything lands” |
| Surfaces | Warp app (macOS/Windows/Linux) · Warp Agent CLI (`warp`) · cloud agents on the Automation Platform; legacy `oz` CLI (“previously shipped as `warp-cli`”) |
| CLI install | `curl -fsSL https://app.warp.dev/download/agent-cli \| bash`, `brew install --cask warp-agent-cli`, PowerShell script; install script self-updates |
| Auth | Browser device-code login (stays signed in); headless: `WARP_API_KEY` env var preferred over `--api-key`; keys created under Settings > Cloud platform > API keys |
| CLI settings | TOML at `~/.warp_cli/settings.toml` (Linux: `~/.config/warp-terminal/cli/`), never synced, hot-reloaded, agent-editable via `/modify-settings` |
| Rules | `AGENTS.md` default (root + cwd, best-effort for edited subdirs); `WARP.md` legacy, wins in same dir; all-caps filename; global `~/.agents/AGENTS.md`; subdir > root > global |
| Skills | `SKILL.md` + YAML frontmatter in `.agents/skills/` (project, to Git root) and `~/.agents/skills/`; `$ARGUMENTS`/`$N` placeholders; bundled `/modify-settings`, `/tui-migrate-setup`, `/agent-add-mcp` |
| Permissions | `agent_decides` / `always_ask` / `always_allow` (+ app `Never`); CLI profiles in `agents.execution_profiles.default`; denylist > allowlist > `agent_decides`; `write_to_pty`, `run_agents`, `ask_user_question` |
| Auto-approve | Per-conversation toggle (`/auto-approve`, `Ctrl+Shift+I`); bypasses your denylist by default (`auto_approve_bypasses_command_denylist = false` to stop); Admin-Panel denylists never bypassed |
| Models | `/model` picker; Auto (`auto`, `auto-efficient`, `auto-genius`, `auto-open`); OpenAI/Anthropic/Google/xAI catalogs with effort suffixes; choice saved as profile `base_model` |
| BYOK | `openai`/`anthropic`/`google` keys in device secure storage (`--set-provider-api-key`); Warp-credit fallback toggle; Auto models always Warp-provided; personal keys local-only |
| Routers | Custom routers in `~/.warp/custom_model_routers/` YAML (+ team-synced); resolve each prompt to a concrete model |
| Sessions | `/conversations`, `--resume CONVERSATION_TOKEN`, `/fork`, `/compact`; conversations save to the Warp account; `!` shell mode + `/natural-language-detection` |
| Planning | Native `/plan` with versioned plan documents; `Ctrl+Shift+P` expand/collapse; “Any update made by the agent **creates a new version**” |
| Orchestration | `/orchestrate`; approval card “Can I start additional agents for this task?”; local children always Warp Agent; cloud children can be Claude Code/Codex; CLI shows status only for cloud children |
| Handoff | `/handoff` forks conversation + snapshots uncommitted changes into a cloud run; blocked while commands/children run; resume finished Warp-Agent runs only |
| Cloud | Triggers (schedule/Slack/Linear/GitHub Actions/API); environments (image + repos + setup); Warp-hosted or managed self-hosting; scheduled agents start fresh each run |
| MCP | App: `~/.warp/.mcp.json` global + `.warp/.mcp.json` project; reads Claude Code/Codex/`~/.agents/.mcp.json` configs; project-scoped never auto-spawn; CLI: `~/.warp_cli/.mcp.json` global-only |
| Memory | Agent Memory: research preview, waitlist, cross-harness stores, async read/write |
| No hooks | No hook or plugin lifecycle documented; context = rules, skills, Codebase Context, MCP, Warp Drive prompts (prompts only in CLI) |

## Official sources

- [Warp overview](https://docs.warp.dev/) · [Agents overview](https://docs.warp.dev/agents) · [Warp Agent CLI](https://docs.warp.dev/agents/cli) · [Profiles & Permissions](https://docs.warp.dev/agents/capabilities/agent-profiles-permissions) · [Model choice](https://docs.warp.dev/agents/inference/model-choice) · [BYOK](https://docs.warp.dev/agents/inference/bring-your-own-api-key) · [Automation Platform](https://docs.warp.dev/platform/overview) · [Warp Agent harness (Factories)](https://docs.warp.dev/factories/harnesses/warp-agent) · [Changelog](https://docs.warp.dev/changelog/2026) · [GitHub: warpdotdev/warp](https://github.com/warpdotdev/warp)
