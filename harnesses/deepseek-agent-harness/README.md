# DeepSeek agent harness

**Verified:** 2026-10-29 against current DeepSeek Harness documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** DeepSeek Harness (`dsh`) Web, Desktop, headless, ACP, SDK, profiles, and plugin runtime  
**Provenance:** Official `deepseek-ai/deepseek-harness` repository unless explicitly marked otherwise

DeepSeek Harness is an open-source, plugin-composed agent harness, not just the DeepSeek model API. It is experimental developer-preview software with compatibility-breaking changes expected. The official safety notice says it has not undergone a security audit and must not be treated as secure or production-ready.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, profiles, and providers](configuration-profiles-and-providers.md) | `$DSH_HOME`, profile composition, patch precedence, models, credentials, logs, and telemetry |
| [Instructions, permissions, and sandbox](instructions-permissions-and-sandbox.md) | Instruction discovery and budgets, permission presets, file-effect policy, and sandbox limits |
| [Agents, skills, and automation](agents-skills-and-automation.md) | Skill roots and formats, subagent providers and limits, experimental agent teams, and scheduling boundaries |
| [MCP, hooks, and plugins](mcp-hooks-and-plugins.md) | MCP transports and naming, experimental hook bridges, Cordis plugins, bundles, and lifecycle behavior |
| [Web, headless, and SDK surfaces](surfaces-web-headless-and-sdk.md) | Web/Desktop, headless output and exit codes, ACP, Python SDK, and `sdk-minimal` differences |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“DeepSeek Harness is a terminal-only CLI.”** `dsh` is the launcher; the primary interactive surface is Web, with a native Desktop shell and headless/ACP/SDK profiles.
- ❌ **“Profile patches always outrank home-wide patches.”** `$DSH_HOME/cordis.patch.yml` is applied after the profile patch and therefore outranks it.
- ❌ **“All profiles use the same permission default.”** Base-backed profiles default new sessions to `workspace-write`; standalone `sdk-minimal` pins `danger-full-access` and has no approval service.
- ❌ **“Workspace-write confines reads, network, and processes.”** It governs file effects; reads and network access are not confined by that mode.
- ❌ **“Nested instruction changes are always detected immediately.”** Instruction refresh is driven by structured filesystem touches and resume, not continuous watching or shell navigation.
- ❌ **“Hook failures reliably stop execution.”** The experimental bridges treat most hook/config failures as non-blocking; only supported blocking outcomes stop an action.
- ❌ **“Local Web/Desktop operation means session data stays local.”** DeepSeek session-log contribution is enabled by default and sends unaccepted log suffixes with subsequent DeepSeek requests unless disabled.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Launcher | `dsh` |
| npm launch | `npx @deepseek-ai/dsh web` |
| Harness home | `$DSH_HOME`, default `~/.dsh` |
| Shared agent home | `$DSH_AGENTS_HOME`, default `~/.agents` |
| Profile | `$DSH_HOME/profiles/<name>/` |
| Profile patch | `$DSH_HOME/profiles/<name>/cordis.patch.yml` |
| Home patch | `$DSH_HOME/cordis.patch.yml` |
| Managed credentials | `$DSH_HOME/.credentials.yaml` |
| Global instructions | `$DSH_HOME/AGENTS.md`, `$DSH_AGENTS_HOME/AGENTS.md` |
| Project instructions | `AGENTS.md`, `CLAUDE.md`, plus additive `.local.md` overlays |
| Project skills | `<projectRoot>/.dsh/skills`, `<projectRoot>/.agents/skills` |
| User skills | `$DSH_HOME/skills`, `$DSH_AGENTS_HOME/skills` |
| Headless | `dsh --profile headless "task"` |
| ACP | `dsh --profile acp` |
| SDK server | `dsh --profile sdk` or `dsh --profile sdk-minimal` |
| Python SDK | `deepseek-harness-sdk`, Python 3.10+ |

## Official sources

- [Project README](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
- [CLI behavior](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
- [Generated CLI help](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cli-help.md)
- [Permission presets](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/interaction/permission-presets/README.md)
- [Sandbox policy](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/sandbox/sandbox-policy/README.md)
- [Agent instructions](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/context/agent-instructions/README.md)
- [Filesystem skills](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/skill-filesystem/README.md)
- [Subagents](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/subagent/subagent/README.md)
- [MCP client](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/mcp/mcp-client/README.md)
- [Hook protocol](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/experimental/hook-protocol/README.md)
- [Python SDK guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/python-sdk.md)
