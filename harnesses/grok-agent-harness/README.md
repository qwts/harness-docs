# Grok Build agent harness

**Verified:** 2026-11-15 against the official Grok Build documentation at docs.x.ai/build and the adjacent Grok Bot docs  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** `grok` CLI/TUI (interactive, headless, ACP), `~/.grok/` config and scopes, permission modes and rules, sandbox profiles, hooks, plugins/skills/marketplaces, MCP, AGENTS.md-family rules, sessions/worktrees, subagents and personas, memory/workflows/background tasks, dashboard, enterprise controls and auth  
**Provenance:** Official xAI sources unless explicitly marked otherwise

Grok Build is xAI’s terminal coding agent — “a powerful and extensible coding agent. Use it via an interactive TUI, headlessly in scripts or bots, or through the Agent Client Protocol (ACP) in other apps.” Installed via `curl -fsSL https://x.ai/cli/install.sh | bash` (or `npm install -g @xai-official/grok`), it keeps everything under `~/.grok/` (`$GROK_HOME`), reads Claude Code and Cursor harness directories through on-by-default compatibility scanners, and ships a Landlock/Seatbelt sandbox, 14 lifecycle hook events, and a five-layer enterprise config stack topped by fail-closed `requirements.toml` pins. The org identities `grokbot`, `grok`, and `grokbuild` normalize to this single harness; Grok Bot (cloud AI teammate) and the Grok assistant app are separate products.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, settings, and models](configuration-settings-and-models.md) | `~/.grok/` layout, five scopes and enterprise layering, project-config limits, `[models]`/`[model.<id>]` BYOK, credential resolution, env vars, Claude/Cursor compat scanners, notable TOML sections |
| [Permissions, sandbox, and plan mode](permissions-sandbox-and-plan-mode.md) | Ask/Auto/Always-approve, Shift+Tab cycle, deny>ask>allow rules, headless `dontAsk`/`acceptEdits`, bypass lock and its root-owned-source rule, sandbox profiles and custom `sandbox.toml`, plan-mode gates and caveats |
| [Hooks, plugins, skills, and MCP](hooks-plugins-skills-and-mcp.md) | 14 hook events and the fail-open script contract, `/hooks-trust`, SKILL.md frontmatter, plugins and marketplaces, MCP transports, `${VAR}`/`{{session_id}}` expansion, project replacement, vendor config merging |
| [Memory, sessions, agents, and runtime](memory-sessions-agents-and-runtime.md) | AGENTS.md-family rules, sessions and rewind, worktrees, subagents and personas, memory/workflows/background tasks, Agent Dashboard |
| [Surfaces: CLI, TUI, headless, and enterprise](surfaces-cli-headless-and-enterprise.md) | Install and four auth methods, TUI keys, modes and slash commands, headless flags and output formats, ACP, network hosts, ZDR, Grok Bot separation |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Configuration lives in `~/.config/grok/` as JSON.”** Settings are TOML under `~/.grok/` (`$GROK_HOME` overrides); project config contributes only `[mcp_servers]`, `[plugins]`, and `[permission]`.
- ❌ **“The sandbox is on by default.”** “Off by default” — profiles (`off`/`workspace`/`devbox`/`read-only`/`strict`) are Landlock/Seatbelt based, and child-network limits are “Enforced on Linux only.”
- ❌ **“Hook failures block the tool call.”** Hooks are fail-open — “Only an explicit `deny` blocks” — and `PreToolUse` is “the only blocking event” of 14; hook timeout default is 5 seconds.
- ❌ **“A project MCP server merges with a same-named user server.”** “A project server with the same name as a user one replaces it entirely.”
- ❌ **“A skill’s `allowed-tools` frontmatter restricts which tools it can run.”** It “Does not grant or restrict tools.”
- ❌ **“Claude Code’s `disableBypassPermissionsMode: "disable"` locks Grok’s always-approve.”** Grok reads that file’s permission rules, MCP allowlists, and marketplace restrictions only; its own lock is `disable_bypass_permissions_mode`, “honored only from root-owned sources.”
- ❌ **“Rewind only changes the conversation.”** “Rewind modifies files on disk — reverted changes are lost unless committed to git.”
- ❌ **“Grok Bot is the cloud/web version of Grok Build.”** Grok Bot is a separate product (“your proactive AI teammate” on cloud computers); the Build overview links it under “Next,” but no Build-session relationship is documented.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Config root | `~/.grok/` (TOML); `$GROK_HOME` overrides; sessions in `~/.grok/sessions/` keyed by working directory |
| Scopes | Environment `GROK_*` → user `config.toml` → project `.grok/config.toml` (MCP/plugins/permission only) → managed → `requirements.toml` pins |
| Enterprise priority | `/etc/grok/managed_config.toml` → `~/.grok/managed_config.toml` → `~/.grok/config.toml` → `~/.grok/requirements.toml` → `/etc/grok/requirements.toml` (highest, fail-closed) |
| BYOK models | `[model.<id>]` with `base_url`, `env_key` (preferred over `api_key`), `api_backend` `chat_completions`\|`responses`\|`messages` |
| Credential resolution | `model.api_key` > `model.env_key` > active session token > `XAI_API_KEY` |
| Permission modes | Ask (default) / Auto (classifier) / Always-approve; `Shift+Tab` cycles Normal → Plan → Auto → Always-approve; headless `dontAsk` / `acceptEdits` |
| Rule order | “deny > ask > allow”; `deny` always wins over `allow`; remembered grants still prompt for `rm`/`git push` |
| Sandbox | Off by default; Landlock (Linux) / Seatbelt (macOS); custom `sandbox.toml` profiles with kernel-enforced glob `deny` |
| Plan mode | Edit-gate only — “bash can still write via redirection”; review not skipped by auto/always-approve; subagents inherit permission mode, not the edit gate |
| Hooks | JSON in `~/.grok/hooks/*.json` / project `.grok/hooks/*.json` (trust-gated); 14 events; exit 2 or `deny` blocks, all else fail-open; timeout default 5 s |
| Skills | `.grok/skills/` (walked up), `~/.grok/skills/`, plugin `skills/`, `[skills] paths`; `user-invocable` default `true` (literal `true` only) |
| MCP | `grok mcp add` (stdio/HTTP); tools named `<server>__<tool>`; `${VAR}` and `{{session_id}}` expansion; OAuth tokens in `~/.grok/mcp_credentials.json` |
| MCP compat | `~/.claude.json`, `.cursor/mcp.json`, `.mcp.json` load below `config.toml`; `[compat.*] mcps = false` disables |
| Instructions | `AGENTS.md`/`Agents.md`/`AGENT.md`/`CLAUDE.md`/`Claude.md`/`CLAUDE.local.md` + `.grok/rules/*.md`; deeper wins; gitignored skipped; no size cap |
| Sessions | Auto-saved to `~/.grok/sessions/`; `/rewind` restores files (destructive); `-s` names new sessions, never resumes |
| Worktrees | `~/.grok/worktrees/<repo>/<name>` from HEAD incl. uncommitted changes; persist until `grok worktree rm`/`gc` |
| Subagents | `general-purpose`/`explore`/`plan` built-ins; `.grok/agents/`, `~/.grok/agents/`; personas are tone/focus overlays |
| Memory & workflows | Cross-session memory off by default (`/flush`, `/memory`, `/dream`); workflows are `.rhai`, on by default; `/loop` expires in 7 days, 50 max |
| Dashboard | `Ctrl+\` / `/dashboard` / `grok dashboard`; inline replies and dispatch; `Ctrl+W` worktree toggle |
| Headless | `grok -p`; `--output-format plain\|json\|streaming-json`; `--no-auto-update` for CI; `--always-approve` alias `--yolo` |
| ACP | `grok agent stdio`; initialize → authenticate (`xai.api_key`\|`cached_token`) → session/new → session/prompt + session/update |
| Auth | Browser OIDC, `--device-auth` (RFC 8628), `auth_provider_command` (`GROK_AUTH_EXPIRED` contract), `XAI_API_KEY` |
| Grok Bot | Separate xAI product (cloud computer teammate); no documented Build-session relationship |
| Install | `curl -fsSL https://x.ai/cli/install.sh \| bash` or `npm install -g @xai-official/grok` |

## Official sources

- [Overview](https://docs.x.ai/build/overview) · [Modes and Commands](https://docs.x.ai/build/modes-and-commands) · [Keyboard Shortcuts](https://docs.x.ai/build/keyboard-shortcuts)
- [Settings](https://docs.x.ai/build/settings) · [Settings Reference](https://docs.x.ai/build/settings/reference) · [Enterprise Deployments](https://docs.x.ai/build/enterprise)
- [Skills, Plugins & Marketplaces](https://docs.x.ai/build/features/skills-plugins-marketplaces) · [MCP Servers](https://docs.x.ai/build/features/mcp-servers) · [Hooks](https://docs.x.ai/build/features/hooks)
- [Permissions](https://docs.x.ai/build/features/permissions) · [Sandbox](https://docs.x.ai/build/features/sandbox) · [Plan Mode](https://docs.x.ai/build/features/plan-mode)
- [Sessions](https://docs.x.ai/build/features/sessions) · [Worktrees](https://docs.x.ai/build/features/worktrees) · [Subagents](https://docs.x.ai/build/features/subagents) · [Background Tasks](https://docs.x.ai/build/features/background-tasks) · [Agent Dashboard](https://docs.x.ai/build/features/dashboard)
- [CLI Reference](https://docs.x.ai/build/cli/reference) · [Headless & Scripting](https://docs.x.ai/build/cli/headless-scripting) · [Terminal Support](https://docs.x.ai/build/cli/terminal-support)
- [Grok Bot overview](https://docs.x.ai/grok-bot/overview) (separate product)
