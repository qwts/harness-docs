# Cline agent harness

**Verified:** 2026-10-29 against current Cline documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** Cline CLI, VS Code and JetBrains extensions, Desktop, SDK, ACP, and Kanban where explicitly documented  
**Provenance:** Official `docs.cline.bot` documentation unless explicitly marked otherwise

Cline has a shared configuration model, but feature availability and file locations still vary by surface. In particular, the CLI and extension hook layouts differ; plugins and agent teams are not extension features; and subagents are documented for CLI and Desktop rather than the IDE extensions.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, rules, and permissions](configuration-rules-and-permissions.md) | Global/project roots, state overrides, rule discovery, conditional activation, Plan/Act, approvals, and `.clineignore` |
| [Agents, skills, and automation](agents-skills-and-automation.md) | Skill discovery and precedence, progressive loading, subagents, agent teams, and approval implications |
| [MCP, hooks, and plugins](mcp-hooks-and-plugins.md) | MCP transports and settings, file-hook layouts, plugin manifests, lifecycle hooks, and isolation limits |
| [CLI, IDE, cloud, and SDK surfaces](surfaces-cli-ide-cloud-and-sdk.md) | Interactive/headless CLI, JSON events, ACP, worktrees, surface support matrix, and SDK packages/tools |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“`.clinerules/` is Cline’s only project configuration layout.”** Current shared configuration uses `.cline/`; `.clinerules/` remains a supported rules layout and is still used for VS Code extension hooks.
- ❌ **“Project skills always override global skills.”** A same-named global skill overrides the project skill.
- ❌ **“A one-shot CLI prompt waits for every tool approval by default.”** Prompt runs default to Act mode with auto-approval enabled; ACP changes the auto-approval default to false.
- ❌ **“The CLI has a built-in shell allow/deny list.”** Current CLI documentation recommends a `PreToolUse` hook for command blocking instead.
- ⚠️ **“Approving `spawn_agent` approves only creation of the subagent.”** It also approves the subagent’s subsequent tool activity; child calls do not independently prompt.
- ❌ **“Plugin worker threads are a security sandbox.”** They provide crash isolation, not a security boundary.
- ❌ **“Subagents and agent teams are two names for one feature.”** They have different availability, coordination models, and state.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| CLI executable | `cline` |
| Global configuration root | `~/.cline/` |
| Application state | `~/.cline/data/` |
| Project configuration | `.cline/` |
| Legacy project rules | `.clinerules/` |
| Workspace rules | `.cline/rules/`, `.clinerules/`, root `AGENTS.md`, `.cursorrules`, `.windsurfrules` |
| Global rules | `~/.cline/rules/`, `~/Documents/Cline/Rules/`, `~/Cline/Rules/`, `~/.agents/AGENTS.md` |
| Project skills | `.cline/skills/` (recommended); documented compatibility locations also exist |
| Global skills | `~/.cline/skills/`, `~/.agents/skills/` |
| CLI MCP settings | `~/.cline/data/settings/cline_mcp_settings.json` |
| CLI workspace hooks | `.cline/hooks/` |
| VS Code workspace hooks | `.clinerules/hooks/` |
| Project plugins | `.cline/plugins/` |
| Global plugins | `~/.cline/plugins/` |
| Team state | `~/.cline/data/db/teams.db` |
| Main SDK package | `@cline/sdk` on Node.js 22+ |

## Official sources

- [Configuration](https://docs.cline.bot/getting-started/config)
- [Cline rules](https://docs.cline.bot/customization/cline-rules)
- [Skills](https://docs.cline.bot/customization/skills)
- [Auto Approve](https://docs.cline.bot/features/auto-approve)
- [CLI reference](https://docs.cline.bot/cli/cli-reference)
- [CLI overview](https://docs.cline.bot/usage/cli-overview)
- [MCP overview](https://docs.cline.bot/mcp/mcp-overview)
- [Plugins](https://docs.cline.bot/customization/plugins)
- [SDK plugin guide](https://docs.cline.bot/sdk/guides/writing-plugins)
- [Hooks](https://docs.cline.bot/customization/hooks)
- [`.clineignore`](https://docs.cline.bot/customization/clineignore)
- [Subagents](https://docs.cline.bot/features/subagents)
- [Agent teams](https://docs.cline.bot/cli/agent-teams)
- [SDK overview](https://docs.cline.bot/sdk/overview)
- [ClineCore](https://docs.cline.bot/sdk/clinecore)
