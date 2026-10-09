# harness-docs

Fact-checked reference documentation for AI agent harnesses: configuration schemas, settings precedence, runtime behavior, and audit logs of common misinformation.

Each harness gets its own folder under `harnesses/` with a README index, topic deep-dives, and a corrections log.

## Harnesses

| Harness | Coverage | Index |
|---|---|---|
| Claude Agent Harness (Claude Desktop + Claude Code) | MCP config, settings hierarchy, hooks, memory, tool search | [harnesses/claude-agent-harness/README.md](harnesses/claude-agent-harness/README.md) |
| Codex Agent Harness (Codex CLI + ChatGPT desktop app + App Server) | config.toml & requirements.toml, precedence & trust, sandbox/approvals, AGENTS.md, subagents, MCP, lifecycle hooks, app-server JSON-RPC, cross-harness migration | [harnesses/codex-agent-harness/README.md](harnesses/codex-agent-harness/README.md) |
| Devin Agent Harness (Devin CLI + Devin Desktop) | config.json / mcp_config.json layers, permission precedence & modes, read_config_from provider imports, rules & AGENTS.md, skills/plugins/subagents/hooks, MCP, ACP, cloud bridge, team settings & system.json | [harnesses/devin-agent-harness/README.md](harnesses/devin-agent-harness/README.md) |
| OpenCode Agent Harness (OpenCode TUI + OpenCode Desktop) | opencode.json merge-order & env overrides, managed/MDM config, permission engine (granular allow/ask/deny), primary agents & subagents, AI SDK + Models.dev provider architecture, MCP + auto-OAuth, custom commands, plugins (hooks/events), AGENTS.md + Claude Code compat | [harnesses/opencode-agent-harness/README.md](harnesses/opencode-agent-harness/README.md) |
| Kiro Agent Harness (Kiro IDE + kiro-cli + Web + Mobile; Crew = separate orchestrator) | `.kiro/` shared config, steering scopes & inclusion modes, AGENTS.md, custom agent configs (`mcpServers`/`includeMcpJson`/`resources`), `.kiro/hooks/*.json` triggers & actions, MCP + `kiro://` installs, surface capability matrix | [harnesses/kiro-agent-harness/README.md](harnesses/kiro-agent-harness/README.md) |

## Conventions

- **Verdict legend** used in all docs: ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED
- Source drafts are audited claim-by-claim against official documentation; corrections are logged per harness in `corrections.md`.
- Unconfirmed claims are marked ❓ UNVERIFIED, never silently included.

## Layout

```
harnesses/
  <harness>/
    README.md        # index, verdict legend, top corrections, quick-reference
    <topic>.md       # verified deep-dives (config, runtime, enterprise controls...)
    corrections.md   # full audit log of source-draft errors
```
