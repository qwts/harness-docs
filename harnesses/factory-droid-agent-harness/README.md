# Factory Droid agent harness

**Verified:** 2026-10-29 against current Factory documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** Droid CLI, Factory App, IDE/ACP integrations, Droid Exec, SDKs, Missions, Delegations, Automations, Environments, and remote execution  
**Provenance:** Official Factory documentation unless explicitly marked otherwise

Factory Droid is a multi-surface agent harness, not just an interactive terminal CLI. Its configuration combines organization policy, runtime overlays, folder/project/user settings, dynamic configuration, and built-in defaults. Interactive sessions, non-interactive execution, delegated work, and hosted compute have different defaults, identity, persistence, and approval behavior.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, precedence, and models](configuration-precedence-and-models.md) | `.factory/` levels, dual precedence, managed policy, local overlays, model policy, BYOK, and credential helpers |
| [Instructions, permissions, and sandbox](instructions-permissions-and-sandbox.md) | Instruction discovery, interaction modes, Autonomy Levels, command rules, sandbox defaults, and security limits |
| [Agents, skills, and automation](agents-skills-and-automation.md) | Custom droids, skills, slash commands, Missions, and Automations |
| [MCP, hooks, plugins, and connectors](mcp-hooks-plugins-and-connectors.md) | MCP configuration and approval, lifecycle hooks, plugin layout, managed connectors, and extension trust |
| [CLI, app, cloud, and SDK surfaces](surfaces-cli-app-cloud-and-sdk.md) | Interactive and IDE surfaces, `droid exec`, JSON-RPC, SDKs, Delegations, Environments, Computers, and Remote Control |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Factory Droid is only a terminal coding agent.”** Current official surfaces include the CLI, Factory App, IDE integrations, headless execution, SDKs, Missions, Delegations, Automations, and cloud or remote compute.
- ❌ **“The settings hierarchy has one universal precedence order.”** Hard controls resolve top-down with Org authoritative; session defaults resolve most-local-first and remain capped by hard controls.
- ❌ **“High Autonomy overrides every policy.”** An effective `ask` still prompts and an effective `block` has no approval path. `--skip-permissions-unsafe` skips prompts but not command blocks.
- ❌ **“Permission rules sandbox every tool and command.”** Native rules match shell-command prefixes; they do not inspect arbitrary program behavior or provide OS isolation.
- ❌ **“A skill’s `allowed-tools` field enforces its tool access.”** It declares intended tools only. Use a custom droid and other policy controls for enforced restrictions.
- ❌ **“Connectors are just another local MCP configuration.”** Connectors use Factory’s managed remote service, are scoped to the Factory user and active organization, and have separate approval behavior.
- ❌ **“A completed `droid exec` process proves the task succeeded.”** Exit `0` means Droid finished; deterministic validation is still required. A permission failure can exit nonzero after earlier edits have already been made.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Interactive CLI | `droid` |
| User settings | `~/.factory/settings.json` (`%USERPROFILE%\.factory\settings.json` on Windows) |
| Project settings | `<git-root>/.factory/settings.json` |
| Same-level local overlay | `settings.local.json` beside `settings.json` |
| Project instructions | `AGENTS.md` and documented compatible filenames |
| Project custom droids | `<repo>/.factory/droids/<name>.md` |
| Personal custom droids | `~/.factory/droids/<name>.md` |
| Skill entry point | `SKILL.md` |
| Workspace commands | `<repo>/.factory/commands/` |
| User MCP | `~/.factory/mcp.json` |
| Project MCP | `<project>/.factory/mcp.json` |
| Hooks | `~/.factory/hooks.json`, `.factory/hooks.json`, and org-managed settings |
| Headless execution | `droid exec "task"`; read-only by default |
| Rule validation | `droid rules check` |
| Python SDK | `droid-sdk`, Python 3.10+ |
| TypeScript SDK | `@factory/droid-sdk`, Node.js 18+ |

## Official sources

- [Droid CLI settings](https://docs.factory.com/droid-cli/settings)
- [Enterprise Controls and managed settings](https://docs.factory.com/enterprise/hierarchical-settings-and-org-control)
- [AGENTS.md](https://docs.factory.com/harness/agents-md)
- [Autonomy Level](https://docs.factory.com/autonomy-and-safety/auto-run)
- [Permission rules](https://docs.factory.com/autonomy-and-safety/permission-rules)
- [Sandbox](https://docs.factory.com/autonomy-and-safety/sandbox)
- [Custom droids](https://docs.factory.com/harness/subagents)
- [Skills](https://docs.factory.com/harness/skills)
- [MCP](https://docs.factory.com/harness/mcp)
- [Hooks](https://docs.factory.com/harness/hooks)
- [Plugins](https://docs.factory.com/harness/plugins)
- [Droid Exec](https://docs.factory.com/droid-exec/overview)
- [Python SDK](https://docs.factory.com/sdk/python)
- [TypeScript SDK](https://docs.factory.com/sdk/typescript)
- [IDE integrations](https://docs.factory.com/ide-integrations)
