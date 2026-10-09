# Devin Agent Harness — CLI & Desktop (Verified)

> **Status:** Written 2026-10-09 directly against official Devin documentation (docs.devin.ai). No source draft was audited for this harness; instead, [corrections.md](corrections.md) audits common misinformation and migration traps seen in third-party coverage.
> **Scope:** The *local* Devin harness — **Devin CLI** (terminal agent) and **Devin Desktop** (the rebranded Windsurf IDE, whose Devin Local agent shares the CLI harness). Devin *cloud* sessions are a different surface; they are covered only where the CLI bridges to them (`devin --cloud`, `devin ssh`, `devin cloud drs`).
> **Intended consumer:** Autonomous agents and deployment orchestrators configuring Devin CLI or Devin Desktop programmatically.

## Verification verdict legend

| Verdict | Meaning |
|---|---|
| ✅ TRUE | Confirmed against official docs (docs.devin.ai) or the installed binary. |
| ⚠️ PARTIAL | Contains truth but is imprecise, overstated, or missing critical nuance. Corrected version given. |
| ❌ FALSE | Refuted by official docs. Do not rely on. |
| 💬 OPINION | Editorial/blog commentary, not harness-specified behavior. Safe as guidance, not as spec. |
| ❓ UNVERIFIED | Plausible but no official source found. Treat as unconfirmed. |

## File map

| File | Contents |
|---|---|
| [config-and-settings.md](config-and-settings.md) | config.json / mcp_config.json locations, permission precedence, permission modes, system.json, team settings, sandbox |
| [imports-and-rules.md](imports-and-rules.md) | `read_config_from` provider-import architecture, AGENTS.md handling, .devin/rules, activation triggers, skills |
| [mcp-and-extensibility.md](mcp-and-extensibility.md) | mcp_config.json schema, `devin mcp` commands, OAuth, plugin format + `devin plugins`, custom subagents, lifecycle hooks |
| [cli-and-desktop-surfaces.md](cli-and-desktop-surfaces.md) | CLI install/auth/commands/modes, cloud bridge (`devin --cloud`, `devin ssh`, `devin cloud drs`), `devin acp`, Devin Desktop (Devin Local, Cascade removal, admin surfaces) |
| [corrections.md](corrections.md) | Audit of common misinformation: config-path traps, Windsurf/Devin naming, MCP file migration, permission precedence myths |

## Top corrections (read first if skimming)

There is no upstream draft for this harness — these are the errors most often repeated about Devin CLI/Desktop, each verified against official docs:

1. **❌ MCP servers do not live in `config.json` anymore.** Since v3000.3 (Local 3.6) they live in dedicated `mcp_config.json` files at each config location; `mcpServers` keys found in main config files are auto-migrated on startup.
2. **⚠️ `.devin/config.json` is not a general project config.** Valid keys are `permissions`, `read_config_from`, `hooks`, and the plugin-governance lists `requiredPlugins`/`optionalPlugins`/`forbiddenPlugins`; model/theme/attribution are user-only (`~/.config/devin/config.json`).
3. **❌ The editor's MCP discovery file is not the agent's MCP file.** Devin Desktop's Cortex-managed agent file is `~/.config/devin/mcp_config.json`; editor discovery uses a separate `~/.codeium/windsurf/mcp_config.json`. They are not interchangeable.
4. **❌ "Cascade is the Devin Desktop agent" is stale.** Cascade was removed in Devin Desktop v3.9.19 (2026-09-08); Devin Local — the same harness as Devin CLI — is now the only bundled agent.
5. **❌ Permission precedence is NOT "user beats project".** Order (highest first): org/team settings → session grants → `.devin/config.local.json` → `.devin/config.json` → user config. And a `deny` always beats every `allow`, regardless of specificity.
6. **⚠️ `devin mcp add` defaults to `local` scope** (per-project, private), not user scope — pass `-s user` or `-s project` explicitly for other scopes.
7. **⚠️ There is no `.devinrules` single-file format.** The single-file legacy format is `.windsurfrules` only; Devin-native rules are the `.devin/rules/*.md` directory format plus `.devin/global_rules.md`.
8. **⚠️ `subagent: true` on a skill silently drops `allowed-tools`/`permissions`.** The skill runs under the `subagent_general` profile with all tools; restrict tools via a custom `agent:` profile instead.
9. **⚠️ Plugin hooks are best-effort and fail open.** A plugin `hooks.json` that fails to load or run does not stop the session — do not rely on plugin hooks for guardrails.
10. **⚠️ `--sandbox` changes the permission model, not just isolation.** Sandbox sessions only offer Autonomous mode (capability prompts instead of command prompts) and fail closed if sandboxing is unavailable.

## Quick-reference: where to write configuration

| Target | Path |
|---|---|
| User config (all projects) | `~/.config/devin/config.json` — Windows: `%APPDATA%\devin\config.json` |
| Project config (committed) | `.devin/config.json` — `permissions`, `read_config_from`, `hooks`, `requiredPlugins`/`optionalPlugins`/`forbiddenPlugins` |
| Project-local overrides (gitignored) | `.devin/config.local.json` (excluded via `.git/info/exclude`) |
| MCP servers | `mcp_config.json` beside each config.json: `~/.config/devin/`, `.devin/`, `.devin/mcp_config.local.json` |
| Global rules | `~/.config/devin/AGENTS.md`, `~/.devin/global_rules.md`, `~/.devin/rules/*.md` |
| Project rules | `AGENTS.md` (16 KiB auto-include cap), `.devin/rules/*.md`, `.devin/global_rules.md` |
| Project hooks | `.devin/hooks.v1.json` (recommended), `"hooks"` key in `.devin/config*.json` |
| Skills (project) | `.agents/skills/<name>/SKILL.md` (recommended), `.devin/skills/`, `.windsurf/skills/`, `.claude/skills/`, `.github/skills/`, `.cognition/skills/` |
| Skills (global) | `~/.config/devin/skills/<name>/SKILL.md` (`%APPDATA%\devin\skills\` on Windows), `~/.agents/skills/`; plugin skills invoke as `/<plugin>:<skill>` |
| Custom subagents (project) | `.devin/agents/<name>.md` or `.devin/agents/<name>/AGENT.md` (also `.agents/agents/`) |
| Custom subagents (global) | `~/.config/devin/agents/` (`%APPDATA%\devin\agents\` on Windows) |
| Machine policy (MDM) | `system.json` in an admin-only system directory — pins auth host and proxy |
| Team/org policy | Devin app Settings → Enterprise → Devin Desktop / CLI team settings — server-side, highest precedence |
| Devin Desktop system hooks | `/Library/Application Support/Devin/hooks.json` (macOS), `/etc/devin/hooks.json` (Linux/WSL), `C:\ProgramData\Devin\hooks.json` (Windows); falls back to legacy `Windsurf` paths |

## Sources consulted (verification, 2026-10-09)

- Official Devin docs (docs.devin.ai): `cli/index`, `cli/essential-commands`, `cli/reference/commands`, `cli/reference/permissions`, `cli/reference/configuration/*` (config-file, global-vs-local, read-config-from), `cli/extensibility/*` (index, rules, skills/*, plugins/overview, mcp/*, hooks/*), `cli/subagents`, `cli/models`, `cli/fusion`, `cli/cloud`, `cli/ssh`, `cli/sandbox`, `cli/enterprise/*` (controls, team-settings, system-config, devin-auth, windsurf-auth), `cli/changelog/stable`, `cli/introducing-devin-cli`, `cli/acp/zed`, `work-with-devin/devin-cli`, `work-with-devin/mcp`, `work-with-devin/devin-mcp`, `onboard-devin/agents-md`, `product-guides/plugins`
- Devin Desktop docs (docs.devin.ai/desktop): `getting-started`, `devin`, `devin-local`, `devin-desktop-faq`, `controls`, `agent-command-center`, `acp`, `cascade/*` (cascade, agents-md, hooks, mcp, memories), `changelog` (v3.9.19), `accounts/*`, `models`, `fusion`, `guide-for-admins`
