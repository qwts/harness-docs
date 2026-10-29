# Goose agent harness

**Verified:** 2026-10-29 against the current `aaif-goose/goose` repository and official documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** goose Desktop, CLI, headless runs, recipes and schedules, subagents, MCP extensions, hooks/plugins, ACP client and server roles, remote servers, roaming, and GDK/SDK surfaces  
**Provenance:** Official goose sources unless explicitly marked otherwise

Goose is a general-purpose local agent harness with Desktop, CLI, and embeddable API surfaces. It is provider-neutral and MCP-extensible, but its interfaces and provider adapters do not have uniform session, tool, approval, or persistence behavior. The current upstream project is [`aaif-goose/goose`](https://github.com/aaif-goose/goose), part of the Agentic AI Foundation at the Linux Foundation; older `block/goose` references are historical.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, precedence, and providers](configuration-precedence-and-providers.md) | Shared files, overrides, credentials, provider classes, extension configuration, and update semantics |
| [Instructions, permissions, and security](instructions-permissions-and-security.md) | `AGENTS.md`/`.goosehints`, per-turn instructions, approval modes, tool policy, detection, adversary review, and security limits |
| [Agents, skills, and recipes](agents-skills-and-recipes.md) | Custom agents, subagents, skills, reusable workflows, subrecipes, schedules, and slash commands |
| [MCP, hooks, plugins, and integrations](mcp-hooks-plugins-and-integrations.md) | Extension lifecycles, allowlists, Code Mode, MCP Apps, plugin packages, and executable lifecycle hooks |
| [CLI, Desktop, ACP, and SDK surfaces](surfaces-cli-desktop-acp-and-sdk.md) | Interactive and headless operation, local and remote ACP, editors, roaming, provider adapters, and SDK scope |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Goose is only a terminal coding agent.”** It has native Desktop, full CLI, headless automation, ACP server/client, editor, recipe, scheduling, and SDK surfaces.
- ❌ **“The CLI and Desktop have separate core provider and extension configuration.”** They share core configuration, although active sessions, interface state, and some credentials or features remain surface-specific.
- ❌ **“Autonomous mode is sandboxed to the current project.”** The default Developer tools can execute commands and modify files with the user’s privileges and access; project instructions are not an OS boundary.
- ❌ **“Smart Approval is deterministic command filtering.”** Read/write classification is a best-effort decision interpreted by the configured LLM provider.
- ❌ **“Every security check fails closed.”** Prompt-injection findings require a user decision, but adversary review fails open, and policy hooks fail open unless `PreToolUse` sets `on_failure: block`.
- ❌ **“Subagent isolation is a filesystem or network sandbox.”** Subagents isolate execution context and lifecycle; inherited extension tools can still act with their granted access.
- ❌ **“ACP support means all providers and clients have feature parity.”** ACP providers cannot currently resume or fork goose sessions, while deprecated CLI providers do not expose goose’s extension ecosystem.
- ❌ **“The Python and Kotlin SDKs embed the complete goose agent.”** Their documented shared bindings cover the provider layer; native Rust crates expose the broader agent orchestration APIs.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Primary config (macOS/Linux) | `~/.config/goose/config.yaml` |
| Primary config (Windows) | `%APPDATA%\Block\goose\config\config.yaml` |
| General precedence | Environment variables → config file → defaults; command, recipe, and subagent overrides add their own rules |
| Secret storage | System keyring preferred; file fallback uses plaintext `secrets.yaml`; provider keys in `config.yaml` are ignored |
| Global context | `~/.config/goose/.goosehints` by default |
| Default context filenames | `AGENTS.md`, then `.goosehints`; configurable with `CONTEXT_FILE_NAMES` |
| Default permission mode | Autonomous (`auto`) |
| Global custom agents | `~/.agents/agents/` |
| Project custom agents | `<project>/.agents/agents/` |
| Global/project skills | `~/.agents/skills/` and `<project>/.agents/skills/` |
| Global/project plugins | `~/.agents/plugins/` and `<project>/.agents/plugins/` |
| Global recipes | `~/.config/goose/recipes/` |
| Project recipes | `<project>/.goose/recipes/` |
| Adversary review | Enabled by `~/.config/goose/adversary.md`; fail-open on reviewer failure |
| Interactive CLI | `goose session` |
| Headless task | `goose run -t "task"` |
| ACP stdio agent | `goose acp` |
| ACP HTTP/WebSocket server | `goose serve`; ACP endpoint `/acp` |

## Official sources

- [Current project repository](https://github.com/aaif-goose/goose)
- [Configuration files](https://goose-docs.ai/docs/guides/config-files)
- [Providers](https://goose-docs.ai/docs/getting-started/providers)
- [Using extensions](https://goose-docs.ai/docs/getting-started/using-extensions)
- [Hints and context files](https://goose-docs.ai/docs/guides/context-engineering/using-goosehints)
- [Permission modes](https://goose-docs.ai/docs/guides/managing-tools/goose-permissions)
- [Tool permissions](https://goose-docs.ai/docs/guides/managing-tools/tool-permissions)
- [Custom agents](https://goose-docs.ai/docs/guides/context-engineering/custom-agents)
- [Subagents](https://goose-docs.ai/docs/guides/context-engineering/subagents)
- [Skills](https://goose-docs.ai/docs/guides/context-engineering/using-skills)
- [Recipes](https://goose-docs.ai/docs/guides/recipes/session-recipes)
- [Hooks](https://goose-docs.ai/docs/guides/context-engineering/hooks)
- [Plugins](https://goose-docs.ai/docs/guides/context-engineering/plugins)
- [ACP agent](https://goose-docs.ai/docs/gdk/acp)
- [SDK](https://goose-docs.ai/docs/gdk/sdk)
- [Security guide](https://goose-docs.ai/docs/guides/security)
