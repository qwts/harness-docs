# Google Antigravity agent harness

**Verified:** 2026-10-29 against current Google Antigravity documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** Antigravity App, Antigravity CLI (`agy`), and standalone IDE/extensions  
**Provenance:** Official `antigravity.google` documentation unless explicitly marked otherwise

Antigravity App and CLI run the same agent and share many customizations, but their settings files, permission defaults, and headless behavior are not identical. The standalone IDE has its own product surface; shared customization formats do not establish full feature parity.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Settings, rules, and permissions](settings-rules-and-permissions.md) | Global/project scopes, App presets, CLI settings, rule discovery, activation and budgets |
| [Agents, skills, and plugins](agents-skills-and-plugins.md) | Custom subagents, workspace modes, skill paths, plugin manifests and components |
| [MCP and hooks](mcp-and-hooks.md) | Transports, timeouts, OAuth, tool filters, lifecycle events and decisions |
| [App, CLI, and IDE surfaces](surfaces-app-cli-and-ide.md) | Projects, CLI/headless operation, soft denials, output formats, surface parity limits |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Antigravity has one permission model with one set of defaults.”** App presets and CLI `agentMode`/`toolPermission` settings are surface-specific.
- ❌ **“All discovered rules replace one another by scope.”** Rules are cumulative; more specific directory instructions win on conflict.
- ❌ **“CLI hooks belong in `settings.json`.”** The CLI retains but never loads a `hooks` key there; use `hooks.json`.
- ❌ **“Headless approval-required tools make every run exit nonzero.”** They are soft-denied by default, and the run can continue and exit 0.
- ⚠️ **“App, CLI, and IDE have identical configuration paths and feature parity.”** They share formats in places, but paths and behavior differ.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| CLI executable | `agy` |
| CLI settings | `~/.gemini/antigravity-cli/settings.json` |
| Project rules | Nested `AGENTS.md`, `GEMINI.md`, `.agents/rules/*.md` |
| Global rules | `~/.gemini/AGENTS.md`, `~/.gemini/GEMINI.md`, `~/.gemini/config/rules/*.md` and documented config variants |
| Workspace subagents | `.agents/agents/<name>.md` or `.agents/agents/<name>/agent.md` |
| Global subagents | `~/.gemini/config/agents/` |
| Workspace skills | `.agents/skills/<skill>/SKILL.md` |
| App global skills | `~/.gemini/config/skills/<skill>/SKILL.md` |
| Workspace plugins | `.agents/plugins/<plugin>/plugin.json` |
| Global plugins | `~/.gemini/config/plugins/<plugin>/plugin.json` |
| Workspace MCP | `.agents/mcp_config.json` |
| Global MCP | `~/.gemini/config/mcp_config.json` |
| Workspace hooks | `.agents/hooks.json` |
| Global hooks | `~/.gemini/config/hooks.json` |
| MCP OAuth tokens | `~/.gemini/antigravity/mcp_oauth_tokens.json` |
| Headless mode | `agy -p` / `--print` / `--prompt` |

## Official sources

- [Settings](https://antigravity.google/docs/settings/)
- [Permissions](https://antigravity.google/docs/permissions/)
- [Rules](https://antigravity.google/docs/rules/)
- [Custom subagents](https://antigravity.google/docs/subagents/)
- [Agent skills](https://antigravity.google/docs/skills/)
- [Plugins](https://antigravity.google/docs/plugins/)
- [MCP](https://antigravity.google/docs/mcp/)
- [Hooks](https://antigravity.google/docs/hooks/)
- [Feature overview](https://antigravity.google/docs/cli/features/)
- [CLI settings](https://antigravity.google/docs/cli/settings/)
- [Headless mode](https://antigravity.google/docs/cli/headless/)
