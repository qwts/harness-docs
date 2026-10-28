# Cursor agent harness

**Verified:** 2026-10-29 against current Cursor documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** Cursor IDE, Cursor CLI, and Cloud Agents  
**Provenance:** Official Cursor documentation unless explicitly marked otherwise

Cursor's local IDE, CLI, cloud agents, and self-hosted cloud machines share some configuration formats but not all runtime sources. In particular, local MCP configuration, personal skill sync, cloud hooks, and environment resolution require surface-specific treatment.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, rules, and permissions](configuration-rules-and-permissions.md) | `.mdc` rules, `AGENTS.md`, permission files and source precedence, Run Modes |
| [Agents, subagents, and skills](agents-subagents-and-skills.md) | Discovery paths, isolated context, nesting, cloud MCP behavior, skill manifests and sync |
| [MCP, hooks, and plugins](mcp-hooks-and-plugins.md) | MCP scopes and capabilities, local/cloud hook limits, two plugin formats |
| [CLI, cloud, and enterprise surfaces](surfaces-cli-cloud-and-enterprise.md) | `agent` CLI, headless/ACP, cloud environment resolution, Builds, self-hosted differences |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Any Markdown file under `.cursor/rules/` is loaded.”** False. Project rules use `.mdc`; plain `.md` files there are ignored.
- ❌ **“Auto-review is a hard sandbox.”** False. It is a best-effort approval/classification guardrail, with sandboxing used where possible.
- ❌ **“Cloud subagents inherit the parent's local MCP configuration.”** False. They use team/cloud MCP configuration.
- ❌ **“Cloud Builds preserve running processes and exported shell state.”** False. Builds preserve disk state.
- ⚠️ **“Cursor supports Agent Plugins exactly as the portable specification is written.”** Cursor has format and variable-expansion differences, including `${CURSOR_PLUGIN_ROOT}`.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Project rules | `.cursor/rules/**/*.mdc` |
| Simpler instructions | `AGENTS.md`, including nested files |
| User permissions | `~/.cursor/permissions.json` |
| Workspace permissions | `<workspace>/.cursor/permissions.json` |
| Permission source precedence | Team admin > `permissions.json` > IDE settings |
| Custom subagents | `.cursor/agents/`; compatibility with `.claude/agents/` and `.codex/agents/` |
| Project skills | `.agents/skills/`, `.cursor/skills/`, plus documented compatibility directories |
| Project MCP | `.cursor/mcp.json` |
| User MCP | `~/.cursor/mcp.json` |
| Workspace hooks | `.cursor/hooks.json` |
| User hooks | `~/.cursor/hooks.json` |
| Agent Plugin manifest | Root `plugin.json` |
| Cursor Plugin manifest | `.cursor-plugin/plugin.json` |
| CLI executable | `agent` |
| Repository cloud environment | `.cursor/environment.json` |

## Official sources

- [Rules](https://cursor.com/docs/rules)
- [Permissions reference](https://cursor.com/docs/reference/permissions)
- [Agent security](https://cursor.com/docs/agent/security)
- [Run Modes](https://cursor.com/docs/agent/security/run-modes)
- [Subagents](https://cursor.com/docs/subagents)
- [Skills](https://cursor.com/docs/skills)
- [MCP](https://cursor.com/docs/mcp)
- [Hooks](https://cursor.com/docs/hooks)
- [Plugins](https://cursor.com/docs/plugins)
- [Using Cursor CLI](https://cursor.com/docs/cli/using)
- [Cloud Agent setup](https://cursor.com/docs/cloud-agent/setup)
- [Cloud Builds](https://cursor.com/docs/cloud-agent/builds)
