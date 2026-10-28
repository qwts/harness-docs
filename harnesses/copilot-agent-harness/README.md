# GitHub Copilot agent harness

**Verified:** 2026-10-29 against current GitHub documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** Copilot CLI, with separately documented Copilot cloud agent and IDE behavior  
**Provenance:** Official GitHub Docs unless a claim is explicitly marked otherwise

Copilot is a family of product surfaces, not one configuration runtime. CLI project files, cloud-agent repository settings, IDE settings, and personal cloud configuration must not be treated as interchangeable.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [CLI configuration and permissions](cli-configuration-and-permissions.md) | `COPILOT_HOME`, trust, tool/path/URL permissions, blanket modes, sandbox boundaries |
| [Instructions, agents, and skills](instructions-agents-and-skills.md) | Instruction discovery, profile deduplication, subagent instruction inheritance, skill locations |
| [MCP, hooks, and plugins](mcp-hooks-and-plugins.md) | CLI MCP precedence, lifecycle hooks, portable and legacy plugin layouts |
| [CLI, cloud, and IDE surfaces](surfaces-cli-cloud-and-ides.md) | Headless CLI, cloud setup workflow, cloud MCP behavior, cross-surface support limits |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Copilot reads `.vscode/mcp.json` in the CLI.”** False. Copilot CLI documents `.mcp.json` and `.github/mcp.json`; `.vscode/mcp.json` is not read by the CLI.
- ❌ **“Allow-all overrides deny rules.”** False. Explicit deny rules still take precedence.
- ❌ **“Cloud-agent MCP is configured by the CLI's project MCP files.”** False. Cloud-agent repository MCP is configured on GitHub and is a distinct configuration surface.
- ⚠️ **“Custom subagents always inherit repository instructions.”** Only when the profile opts in with `include-custom-instructions: true`.
- ⚠️ **“All applicable Copilot instruction files have a documented precedence order.”** They are combined; GitHub documents no general order and recommends avoiding conflicts.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| CLI home | `~/.copilot/`; relocate with `COPILOT_HOME` |
| Persistent trusted folders | `~/.copilot/config.json`, `trustedFolders` |
| User instructions | `~/.copilot/copilot-instructions.md` and `~/.copilot/instructions/**/*.instructions.md` |
| Repository instructions | `.github/copilot-instructions.md`, `.github/instructions/**/*.instructions.md`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` |
| Custom agents | Project `.github/agents/`; user `~/.copilot/agents/` |
| Project skills | `.github/skills/`, `.claude/skills/`, or `.agents/skills/` |
| Personal skills | `~/.copilot/skills/` or `~/.agents/skills/` |
| User MCP | `~/.copilot/mcp-config.json` |
| Project MCP | `.mcp.json` and `.github/mcp.json` |
| Repository hooks | `.github/hooks/*.json` |
| User hooks | `~/.copilot/hooks/*.json` |
| Cloud setup | `.github/workflows/copilot-setup-steps.yml` on the default branch |
| Broad bypass | `--allow-all` / `--yolo`; deny and enterprise restrictions still apply |

## Official sources

- [Configuring Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/configure-copilot-cli)
- [Adding custom instructions for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Creating custom agents for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Adding agent skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- [Adding MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
- [Using hooks](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks)
- [Creating plugins](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)
- [MCP across Copilot surfaces](https://docs.github.com/en/copilot/concepts/customizations/mcp)
- [Customizing the cloud-agent environment](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/customize-the-agent-environment)
