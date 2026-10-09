# Kiro Agent Harness (Kiro IDE + kiro-cli + Web + Mobile)

> Verified 2026-10-09 against [kiro.dev/docs](https://kiro.dev/docs/) (docs updated 2026-10-07). Covers AWS's Kiro agent harness: one unified agent runtime behind four surfaces — IDE (Code OSS editor), `kiro-cli` terminal agent, Web agent, and Mobile — sharing `.kiro/` project configuration. **Crew is a separate orchestrator** (not a fifth front end): it drives selectable agent backends incl. `kiro-cli`, Claude Code, Codex, and others.

**Status**: complete, verified. **Provenance**: written directly against official docs — no source draft, so `corrections.md` audits common misinformation (mostly stale Amazon Q-era facts).

## Verdict legend

| Verdict | Meaning |
|---|---|
| ✅ TRUE | Claim verified against official docs or source. |
| ⚠️ PARTIAL | Claim is real but overstated, underspecified, or only conditionally true. |
| ❌ FALSE | Claim contradicts official docs. |
| 💬 OPINION | Unverifiable preference, not a fact. |
| ❓ UNVERIFIED | Could not be confirmed from official sources. |

## File map

| File | Contents |
|---|---|
| [config-and-steering.md](config-and-steering.md) | `.kiro/` layout, steering files (workspace/global/team), inclusion modes (`always`/`fileMatch`/`manual`/`auto`), AGENTS.md support, Kiroignore, Configuration Sync |
| [agents-and-permissions.md](agents-and-permissions.md) | custom agent JSON configs, `mcpServers`/`includeMcpJson`/`resources`, permission & surface-specific behavior |
| [mcp-hooks-and-extensibility.md](mcp-hooks-and-extensibility.md) | MCP config (workspace/user `mcpServers`, `kiro://` install links, tool validation), hooks (`.kiro/hooks/*.json` schema, triggers, command/agent actions, confirm), powers/skills/sub-agents |
| [surfaces-cli-and-web.md](surfaces-cli-and-web.md) | the 4 surfaces on one harness (+ Crew orchestrator), capability matrix, kiro-cli versions & Q migration, ACP |
| [corrections.md](corrections.md) | misinformation audit: Q Developer vs Kiro naming, hook-format migration, steering vs AGENTS.md, MCP scoping |

## Top corrections (read first if skimming)

1. **❌ Kiro is not just the IDE — but Crew is not part of the harness.** One runtime powers four front ends: IDE, `kiro-cli`, Web, Mobile. Crew is a separate orchestrator that selects agent backends (`kiro-cli`, Claude Code, Codex…); `.kiro/` config is shared only by the four.
2. **⚠️ Steering ≠ AGENTS.md.** AGENTS.md is always-included (no frontmatter modes, discovered across subdirs); steering `.md` files support inclusion modes. Both coexist.
3. **❌ Agent hooks aren't chat-only config anymore.** `.kiro/hooks/*.json` standalone files replaced the embedded formats in IDE 1.0 / CLI 3.0; `kiro-cli agent migrate` converts CLI 2.x.
4. **⚠️ MCP config is per-scope, not one global file.** Workspace + user MCP JSON, plus per-agent `mcpServers` gated by `includeMcpJson`.
5. **⚠️ `kiro://` install links are not silent installs.** Kiro shows a confirmation dialog (command + env var names, values hidden) before writing config.
6. **⚠️ Steering inclusion modes need CLI V3.** V1/V2 only auto-load `always` files; `fileMatch`/`manual`/`auto` are V3+.
7. **⚠️ Global steering doesn't reach Web/Mobile.** `~/.kiro/steering/` is unreadable from the cloud sandbox — use Configuration Sync upload instead.
8. **⚠️ Hooks run on tool events, not just file saves.** Triggers are PascalCase events (`PostFileSave`, `PreToolUse`, `PostToolUse`, `Stop`…); matcher is a regex against tool name or file path.
9. **⚠️ MCP tool names have hard rules.** ≤64 chars incl. prefix, `^[a-zA-Z][a-zA-Z0-9_]*$`, non-empty description — invalid tools are excluded, not warned-and-used.

## Quick reference

| Surface | Path / handle |
|---|---|
| Project config root | `.kiro/` (travels with the repo across all surfaces) |
| Workspace steering | `.kiro/steering/*.md` (+ frontmatter `inclusion:`) |
| Global steering | `~/.kiro/steering/` (IDE/CLI only; sync to cloud via Configuration Sync) |
| Rules fallback | `AGENTS.md` at repo root + subdirs + `~/.kiro/steering/AGENTS.md` |
| Hooks | `.kiro/hooks/*.json` (schema `version: "v1"`) |
| MCP config | workspace `.kiro/settings/mcp.json` + user `~/.kiro/settings/mcp.json` ⚠️ (palette: "Kiro: Open workspace/user MCP config (JSON)") |
| Agent configs | `.kiro/agents/<name>.json` or `.kiro/agents/<name>.md` (`name`, `description`, `mcpServers`, `includeMcpJson`, `resources`) |
| CLI | `kiro-cli` (v3 current; `kiro-cli agent migrate` for 2.x hooks) |
| Install scheme | `kiro://` MCP install links w/ confirmation dialog |
| Secret guard | `.kiroignore` file patterns (IDE + CLI V3 only — not Web/Mobile) |
| Cloud reuse | Configuration Sync (uploads local steering/config to cloud sessions) |

## Sources

- https://kiro.dev/docs/ — unified-harness model, surfaces, feature list
- https://kiro.dev/docs/steering/ — steering scopes, inclusion modes, AGENTS.md, team steering
- https://kiro.dev/docs/mcp/ — MCP config, agent `mcpServers`/`includeMcpJson`, `kiro://` links, validation rules
- https://kiro.dev/docs/hooks/ — `.kiro/hooks/*.json` schema, triggers, actions, confirm/confirmCommand, migration
