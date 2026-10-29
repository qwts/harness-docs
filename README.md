# harness-docs

Fact-checked reference documentation for AI agent harnesses: configuration schemas, settings precedence, runtime behavior, and audit logs of common misinformation.

Each harness gets its own folder under `harnesses/` with a README index, topic deep-dives, and a corrections log.

The [registry-backed coverage index](harnesses/README.md) tracks active unique harnesses from `qwts-agent-org` and identifies documented versus pending coverage.

## Harnesses

| Harness | Coverage | Index |
|---|---|---|
| Claude Agent Harness (Claude Desktop + Claude Code) | MCP config, settings hierarchy, hooks, memory, tool search | [harnesses/claude-agent-harness/README.md](harnesses/claude-agent-harness/README.md) |
| Codex Agent Harness (Codex CLI + ChatGPT desktop app + App Server) | config.toml & requirements.toml, precedence & trust, sandbox/approvals, AGENTS.md, subagents, MCP, lifecycle hooks, app-server JSON-RPC, cross-harness migration | [harnesses/codex-agent-harness/README.md](harnesses/codex-agent-harness/README.md) |
| GitHub Copilot Agent Harness (Copilot CLI + cloud agent + IDE surfaces) | CLI trust and permissions, instructions, custom agents and subagents, skills, project/user MCP precedence, hooks, Agent Plugins, cloud setup and MCP differences | [harnesses/copilot-agent-harness/README.md](harnesses/copilot-agent-harness/README.md) |
| Cursor Agent Harness (Cursor IDE + CLI + Cloud Agents) | rules and AGENTS.md, permission sources and Run Modes, subagents, skills, MCP, hooks, Agent/Cursor Plugins, CLI, cloud environments and Builds | [harnesses/cursor-agent-harness/README.md](harnesses/cursor-agent-harness/README.md) |
| Cline Agent Harness (CLI + IDE extensions + Desktop + SDK/Kanban) | global/project configuration, rules and conditional activation, Plan/Act and auto-approval, skills, subagents and agent teams, MCP, file hooks, plugins, headless JSON, ACP and SDK | [harnesses/cline-agent-harness/README.md](harnesses/cline-agent-harness/README.md) |
| DeepSeek Agent Harness (`dsh` Web + Desktop + headless + ACP/SDK) | plugin-composed profiles and patch precedence, providers and credentials, instructions, permission presets and file sandbox, skills, subagents, MCP, experimental hooks, plugins and Python SDK | [harnesses/deepseek-agent-harness/README.md](harnesses/deepseek-agent-harness/README.md) |
| Devin Agent Harness (Devin CLI + Devin Desktop) | config.json / mcp_config.json layers, permission precedence & modes, read_config_from provider imports, rules & AGENTS.md, skills/plugins/subagents/hooks, MCP, ACP, cloud bridge, team settings & system.json | [harnesses/devin-agent-harness/README.md](harnesses/devin-agent-harness/README.md) |
| Meta Muse Code Agent Harness (`muse` CLI + server + SDK) | settings and trust, instruction selection, memory, permission profiles, fail-closed sandbox, subagents and worktrees, hooks, MCP, workflows, headless CI and SDK | [harnesses/muse-agent-harness/README.md](harnesses/muse-agent-harness/README.md) |
| Google Antigravity Agent Harness (App + CLI + IDE) | global/project settings, permission presets and rules, cumulative instructions, subagents, skills, plugins, MCP, lifecycle hooks, CLI headless mode and surface differences | [harnesses/antigravity-agent-harness/README.md](harnesses/antigravity-agent-harness/README.md) |
| OpenCode Agent Harness (OpenCode TUI + OpenCode Desktop) | opencode.json merge-order & env overrides, managed/MDM config, permission engine (granular allow/ask/deny), primary agents & subagents, AI SDK + Models.dev provider architecture, MCP + auto-OAuth, custom commands, plugins (hooks/events), AGENTS.md + Claude Code compat | [harnesses/opencode-agent-harness/README.md](harnesses/opencode-agent-harness/README.md) |
| Kiro Agent Harness (Kiro IDE + kiro-cli + Web + Mobile; Crew = separate orchestrator) | `.kiro/` shared config, steering scopes & inclusion modes, AGENTS.md, custom agent configs (`mcpServers`/`includeMcpJson`/`resources`), `.kiro/hooks/*.json` triggers & actions, MCP + `kiro://` installs, surface capability matrix | [harnesses/kiro-agent-harness/README.md](harnesses/kiro-agent-harness/README.md) |

## Conventions

- **Verdict legend** used in all docs: ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED
- Source drafts are audited claim-by-claim against official documentation; corrections are logged per harness in `corrections.md`.
- Unconfirmed claims are marked ❓ UNVERIFIED, never silently included.

## Layout

```
harnesses/
  README.md        # registry-backed active-harness coverage index
  <harness>/
    README.md        # index, verdict legend, top corrections, quick-reference
    <topic>.md       # verified deep-dives (config, runtime, enterprise controls...)
    corrections.md   # full audit log of source-draft errors
```
