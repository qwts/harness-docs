# Claude Agent Harness — Configuration Reference (Verified)

> **Status:** Reorganized + fact-checked 2026-10-09 against official Claude Code / MCP documentation.
> **Provenance:** Derived from a Gemini-produced draft ("Architecture and Configuration Schema of the Claude Agent Harness"), which was audited claim-by-claim. False and unsupported claims were corrected or removed; surviving claims are marked with a verdict.
> **Intended consumer:** Autonomous agents and deployment orchestrators configuring Claude Desktop and Claude Code programmatically.

## Verification verdict legend

| Verdict | Meaning |
|---|---|
| ✅ TRUE | Confirmed against official docs (docs.claude.com / code.claude.com / modelcontextprotocol.io) or multiple independent sources. |
| ⚠️ PARTIAL | Contains truth but is imprecise, overstated, or missing critical nuance. Corrected version given. |
| ❌ FALSE | Refuted by official docs or multiple independent sources. Do not rely on. |
| 💬 OPINION | Editorial/blog commentary, not harness-specified behavior. Safe as guidance, not as spec. |
| ❓ UNVERIFIED | Plausible but no official source found. Treat as unconfirmed. |

## File map

| File | Contents |
|---|---|
| [desktop-config.md](desktop-config.md) | Claude Desktop: config paths, schema, lifecycle, env inheritance, Windows quirks |
| [cli-mcp-config.md](cli-mcp-config.md) | Claude Code: MCP scopes, precedence, JSON schema, transports, `claude mcp` commands, Desktop import |
| [cli-settings-precedence.md](cli-settings-precedence.md) | Settings hierarchy (5 tiers), enterprise managed settings, managed-mcp.json, env variables |
| [runtime-behavior.md](runtime-behavior.md) | Hooks as kill switches, layered prompts, memory system, tool search, dynamic tool updates |
| [corrections.md](corrections.md) | Full audit log: every false/overstated claim from the source draft, with the correction |

## Top corrections (read first if skimming)

The source draft was **mostly accurate** on paths, JSON schemas, and settings precedence, but contained these defects:

1. **❌ MCP scope precedence is REVERSED in the draft.** The draft claims User scope overrides Project scope for a same-named server. Official behavior: **local > project > user** (then plugin-provided, then claude.ai connectors). The project-scope definition wins over user scope.
2. **❌ `claude mcp add-from-claude-desktop` does NOT write imports to global user scope by default.** It writes to **local scope** (per-project, in `~/.claude.json` under the project path). `--scope user` is an explicit opt-in for all-projects availability.
3. **❌ A transport `type` field is NOT mandatory for stdio servers.** A typeless JSON entry is read as a stdio server. `type` is required only for remote transports (`http`, `sse`, `ws`) — a `url` without `type` is a config error. The reserved `"type": "sdk"` really is rejected outside SDK host applications (this part of the draft was correct).
4. **❌ The "XML citation tags / decay signal" memory mechanism appears fabricated.** Real auto memory is plain Markdown with YAML frontmatter (`name`, `description`, `type`, `modified`) in `~/.claude/projects/<project>/memory/`. No XML citation or decay mechanics exist in official docs.
5. **⚠️ Linux Desktop path is misleading.** Anthropic ships Claude Desktop for macOS and Windows only. `~/.config/Claude/claude_desktop_config.json` applies only to unofficial/Electron-wrapped builds. On Linux, Claude Code (CLI) is the supported MCP surface.
6. **⚠️ "Strictly request-response" is outdated.** MCP servers can push `list_changed` notifications without restart, and Claude Code now supports opt-in **channels** where a server can push messages into a session.
7. **💬 DAG orchestration, terminal states, and the performance/economics section are blog opinion**, not harness-specified behavior. Retained in [runtime-behavior.md](runtime-behavior.md) as guidance only, clearly labeled.

## Quick-reference: where to write configuration

| Target | Path |
|---|---|
| Claude Desktop MCP servers (macOS) | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Desktop MCP servers (Windows) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Claude Code MCP, local + user scope | `~/.claude.json` (local = per-project entry; user = top-level) |
| Claude Code MCP, project scope | `.mcp.json` at project root (commit to share; approval-gated) |
| Claude Code shared project settings | `.claude/settings.json` |
| Claude Code private project settings | `.claude/settings.local.json` (auto-gitignored) |
| Enterprise settings (managed) | `managed-settings.json` — macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\` |
| Enterprise MCP exclusivity | `managed-mcp.json` (same managed locations) |

## Sources consulted (verification, 2026-10-09)

- Official Claude Code docs: `docs.claude.com/en/docs/claude-code/mcp`, `.../memory`, `.../hooks`, `.../settings`, `code.claude.com/docs/en/managed-mcp`
- MCP official docs: `modelcontextprotocol.io` (Connect to local MCP servers)
- Independent cross-checks: builder.io, scalar.com, agentcat.com, mcpbundles.com, thepromptshelf.dev, unipile.com, ego.app, systemprompt.io, GitHub issues on anthropics/claude-code
