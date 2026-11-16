# ZCode Agent Harness (Z.AI ZCode)

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; desktop app v3.18.2 “· Latest”, changelog release 2026-11-13; Memory requires ≥ v3.6.4; tray-close default since v3.4.0)

ZCode is “an Agentic Development Environment (ADE) built to bring GLM-5.3 into real coding workflows … a stable desktop experience for planning, coding, reviewing, and iterating across complex development tasks.” Its primary agent is “ZCode Agent is the default self-developed agent in ZCode,” which is “deeply adapted for the GLM-5.3 model family” (stable 1M context; thought-level toggle surfacing GLM-5.3's flexible effort). Surfaces: the desktop workspace (macOS Apple Silicon/Intel, Windows x64/ARM64, Linux x64/ARM64), the built-in desktop browser panel, mobile **Remote Control**, the **Bot Channel** (WeChat/Feishu), and remote workspaces over SSH/WSL/Docker. This reference scopes to the ZCode Agent/ADE — not the GLM model family or generic Z.AI API access. No CLI or headless surface is documented; automation runs “on your machine.” Registry alias: roster `zcode` under `qwts-zcode-agent`.

Version anchors: latest download **v3.18.2** (2026-11-13: voice input, Worktree support, Design capability, session management tool, remember-Full-Access on authorization); Project Memory needs ≥ v3.6.4; Windows hide-to-tray default since v3.4.0. Credentials and account-specific secrets are excluded throughout.

**Verdict legend:** ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## In this folder

| File | Covers |
|---|---|
| [installation-and-connection.md](installation-and-connection.md) | Desktop install (v3.18.2), first launch + migration wizard, Connect Z.ai/BigModel/API key, coding vs general vs Anthropic endpoints, trial quota (5-day, 8M tokens/day), team plans, proxy semantics (no HTTP_PROXY read), Windows shell/tray, `zcode://` login |
| [configuration-context-and-skills.md](configuration-context-and-skills.md) | AGENTS.md two-source rule (no merge/scan/@import, CLAUDE.md onboarding-only), context entries (@/#//$), Project Memory (off by default, four info kinds, no browse/clear), skills (`SKILL.md`, 1024-char description cap, metadata budget), commands (`/goal`, `/compact`, `~/.zcode/commands`), subagents (`~/.zcode/agents/<name>.md`, Explore read-only, custom = Beta), context-window/compaction/thinking-effort rules |
| [permissions-and-execution-modes.md](permissions-and-execution-modes.md) | Four execution modes (Ask before changes default; Shift+Tab), Allow/Always Allow/Reject/Always Reject + session/project granularity, 5-minute question timeout with auto-continue, hook events/sources and the project-hooks block |
| [sessions-goals-and-agent-features.md](sessions-goals-and-agent-features.md) | Workspace entry, work-outside-project mode, thought level, Goal Mode (`/goal` auto-rounds), side conversations (`/side`, `/btw`) and forking rules, browser automation plugin, Worktree (release note), wiki/edit-history/shares |
| [integrations-automation-and-remote.md](integrations-automation-and-remote.md) | MCP (stdio/HTTP/SSE, OAuth one-click, scopes, import), plugin store (Claude Code marketplace preloaded, GitHub-served catalog), scheduled tasks (local machine, must stay awake), idle-time tasks (free, Coding-Plan-gated, runs once), Remote Control + Bot Channel boundaries, Remote Development (SSH/WSL/Docker), verified no-CLI/API/webhook absence |
| [corrections.md](corrections.md) | Claim-by-claim audit log (33 rows: 23 ❌ / 10 ⚠️) |

## Top corrections

1. **“ZCode is just a GLM chat wrapper.”** ❌ “ZCode is a new Agentic Development Environment (ADE)” with a first-party Agent, workspace/task management, terminal, Git, live browser preview, and automations.
2. **“ZCode reads your terminal's GLM environment variables.”** ❌ “Terminal environment variables and the ZCode desktop model setup are separate entry points, so configuration is not synced automatically”; it also “does not read HTTP_PROXY or similar environment variables.”
3. **“ZCode ships a CLI or headless mode.”** ❌ Only the desktop app is documented; tasks “run on your machine, so nothing triggers while it's asleep or the app is closed,” and Remote/Bot are control surfaces that “never creates a runtime of its own”.
4. **“Project-level hooks run.”** ❌ “Project-level hooks are not executed in the current version … ignored as a whole — regardless of hooks.enabled — and logged as `config_project_hooks_ignored`.”
5. **“CLAUDE.md is read continuously.”** ❌ “CLAUDE.md is not continuously read by ZCode Agent at runtime” — onboarding migration source only.
6. **“Nested AGENTS.md files merge.”** ❌ “It does not merge multiple AGENTS.md files across directory levels. It also does not scan child directories, expand @import / @include.”
7. **“Scheduled tasks run in the cloud.”** ❌ “Your computer must be awake … a missed one-time run is recorded as skipped rather than replayed.”
8. **“Memory is on by default.”** ❌ “Memory is off by default and requires ZCode v3.6.4 or later,” and it applies to new sessions.
9. **“Over-long skill descriptions get truncated.”** ❌ “Going over drops the whole skill … rather than truncating it” (1024-char cap).
10. **“The Anthropic endpoint bills prepaid balance.”** ❌ It draws from balance “only for accounts that have never purchased a Coding Plan and have received additional allowlisting”; use the general OpenAI endpoint.
11. **“Idle-time tasks consume plan quota.”** ❌ “The run is free and doesn't consume your subscription plan quota” — but they need an active Coding Plan (Start Plan and API-key mode unsupported) and run once.
12. **“Remote Control / bots run a separate ZCode runtime.”** ❌ “The phone is a control surface: it doesn't sync code and never creates a runtime of its own”; instructions execute on the desktop machine.

## Quick reference

| Area | Fact |
|---|---|
| Product | “Agentic Development Environment (ADE)” for GLM-5.3; desktop app free, model access via plan/key; “deeply tuned … around GLM-5.3” |
| Install | macOS (dmg, Apple Silicon/Intel), Windows (x64/ARM64), Linux (x64/ARM64, AppImage/DEB/RPM); v3.18.2 latest (2026-11-13) |
| Connect | Connect Z.ai / Connect BigModel / Use API Key; Coding endpoint `/api/coding/paas/v4` vs general `/api/paas/v4` vs Anthropic `/api/anthropic`; trial 8M tokens/day × 5 days (GLM-5.3 3M + Flash 5M) |
| Plans | GLM Coding Plan Lite/Pro/Max metered in credits + weekly limit (Pro 6×, Max 14× Lite); in-app purchase; Team Plan entries per org; seats managed on BigModel, not in client |
| Third-party | Anthropic, OpenRouter, Moonshot, OpenAI, DeepSeek-compatible (both protocols); provider `options` accepts only `apiKey`/`baseURL`/`apiKeyRequired`/`headers` — everything else silently ignored |
| Context | New models default to 200K window; `[1m]` IDs fixed at 1M; Coding Plan windows server-issued (edits restored on sync); compaction reserves ~34K (128K→~94K, 1M→~966K), no threshold switch |
| Thinking | GLM-5.3 low/high/max (default max); third-party levels may 400 on unofficial endpoints; `reasoning_effort` (OpenAI protocol) vs `thinking`/`effort` (Anthropic protocol) |
| Instructions | `~/.zcode/AGENTS.md` + workspace `AGENTS.md` (global first, workspace primary); no multi-level merge, no child-dir scan, no `@import`; CLAUDE.md = onboarding migration only |
| Context entries | `@` files/folders, `#` past conversations, `/` commands, `$` skills, attachments; oversized pastes become attachments |
| Memory | Off by default (v3.6.4+); User/Feedback/Project/Reference; “remember that …” / “forget what I said about …”; no browse/clear UI; main conversation only (subagents excluded) |
| Skills | `~/.zcode/skills/<name>/SKILL.md`; name+description required; description cap 1024 (over → dropped); body >100KB truncated; per-turn metadata ≤250 chars with fixed budget; no standalone marketplace |
| Commands | Built-ins `/goal`, `/compact`; custom `~/.zcode/commands/*.md` (User/Workspace scope); import from Claude Code |
| Subagents | Built-ins `general-purpose` + read-only `Explore`; custom Beta → `~/.zcode/agents/<name>.md` (name/description required, camelCase frontmatter); built-ins not editable |
| Modes | Ask before changes (default) / Edit automatically / Plan / Full access; Shift+Tab cycles; Allow · Always Allow · Reject · Always Reject (+session/project scopes); v3.18.2 adds mode selection during authorization + remember Full Access |
| Questions | 5-min default timeout → “Unanswered, auto-continued”; any interaction stops countdown permanently; permission requests and plan approvals always wait |
| Hooks | SessionStart/UserPromptSubmit/PreToolUse/PermissionRequest/PostToolUse/PostToolUseFailure/Stop (block loops ≤3); user `~/.zcode/cli/config.json` (`hooks.enabled: true`) + plugin `hooks/hooks.json`; project hooks ignored (`config_project_hooks_ignored`) |
| Goal Mode | `/goal` set/replace/pause/resume/clear; auto-check per round until met; one goal per session |
| Sessions | Side conversations (`/side`, `/btw`; desktop-only, one per task/window, unrecoverable); fork from completed assistant messages only (no disk rollback); edit last user turn only |
| Browser | Official Browser Use plugin, on by default, desktop-only panel; element picker; PNG/PDF export, device presets (v3.18.2) |
| MCP | User/Workspace scope; stdio/HTTP/SSE; full-config JSON accepts both styles; OAuth via system browser (remote only); import from Claude Code/Codex CLI/OpenCode |
| Plugins | Store = Public (GitHub-served) + Personal marketplaces; Claude Code marketplace preloaded; bundles skills/commands/agents/MCP/hooks |
| Automations | Scheduled: Hourly→Custom repeat rules, run on your machine (Keep awake switch), indefinite until Ends date, no run-N-times field; idle-time: free, single-run queue, Coding Plan required, batch rollout |
| Remote | Phone Remote Control = control surface of the desktop window (one page at a time); Bot Channel WeChat/Feishu (DingTalk/Discord/WeCom later); Remote Development SSH/WSL/Docker (POSIX shell required, no native Windows hosts) |
| No CLI | No CLI/headless/API/webhook documented; tasks run locally on the desktop machine |

## Official sources

- [Welcome](https://zcode.z.ai/en/docs/welcome) · [Install](https://zcode.z.ai/en/docs/install) · [Connect Models](https://zcode.z.ai/en/docs/configuration) · [ZCode Agent](https://zcode.z.ai/en/docs/agents) · [Safety Confirmation](https://zcode.z.ai/en/docs/safety-confirm) · [Hooks](https://zcode.z.ai/en/docs/hooks) · [Automations](https://zcode.z.ai/en/docs/automations) · [Remote Control](https://zcode.z.ai/en/docs/remote-control) · [FAQ (Q&A)](https://zcode.z.ai/en/docs/qa) · [Changelog](https://zcode.z.ai/en/changelog)
