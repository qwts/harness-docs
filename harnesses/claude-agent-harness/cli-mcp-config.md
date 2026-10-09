# Claude Code — MCP Configuration: Scopes, Schema, Commands (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

## Configuration scopes and file paths

| Scope | Target file | Behavior | Verdict |
|---|---|---|---|
| Local (**default** for `claude mcp add`) | `~/.claude.json`, under the current project's path entry | Private to you, active only in the project where added. | ✅ TRUE — **the draft omitted this scope as the default** |
| User | `~/.claude.json`, top-level | Global defaults across all projects on the machine. If `CLAUDE_CONFIG_DIR` is set, Claude Code reads `.claude.json` from inside that directory instead. | ✅ TRUE (CLAUDE_CONFIG_DIR confirmed) |
| Project | `.mcp.json` at project root (+ `.claude/settings.json` for settings) | Committed to version control; shared with the team. **Approval-gated**: new servers show `⏸ Pending approval` in interactive sessions until approved; approvals committed in the repo's own settings are ignored until the workspace is trusted. | ✅ TRUE |
| Local settings | `.claude/settings.local.json` | Private project overrides; automatically gitignored. | ✅ TRUE |
| Managed (enterprise) | `managed-settings.json` and `managed-mcp.json` | See [cli-settings-precedence.md](cli-settings-precedence.md). | ✅ TRUE |

## ❌ Scope precedence for same-named servers — CRITICAL CORRECTION

The source draft claims: *"If the User Scope and the Project Scope define the same server alias, the User Scope definition overrides the project-level definition."* **This is backwards.**

✅ TRUE (corrected) — Precedence when the same server name is defined in multiple scopes:

```
local > project > user > plugin-provided > claude.ai connectors
```

- The highest-precedence entry is used **whole** — fields are **not merged** across scopes.
- Claude Code connects to one instance per server name and warns about the conflict in `claude mcp list` and `/mcp`.
- OAuth sign-ins are stored per endpoint: authenticating in one project doesn't carry to a project where a different definition loads.

## JSON schema for CLI MCP entries

| Field | Requirement | Verdict |
|---|---|---|
| `command`, `args`, `env` | For local stdio servers — same shape as Desktop | ✅ TRUE |
| `type` | **Inferred as `stdio` when omitted.** Required only for remote transports: an entry with `url` but no `type` is a configuration error (`add "type": "http"` / `"sse"` / `"ws"`). | ⚠️ PARTIAL in draft — the draft claimed `type` is mandatory for *every* server; it is not for stdio. |
| `url` | For remote transports, replaces `command` array. | ✅ TRUE |
| `headers`, `headersHelper` | For remote servers (auth tokens). | ✅ TRUE (draft omitted) |
| `alwaysLoad` | Forces eager loading of a server's tools even under tool search. | ✅ TRUE (draft omitted) |

### Transport types

| Type | Use | Verdict |
|---|---|---|
| `stdio` | Local processes on the host (default inference) | ✅ TRUE |
| `http` (JSON alias `streamable-http`) | Recommended for remote/cloud servers | ✅ TRUE |
| `sse` | Deprecated legacy remote transport; HTTP is preferred. `--transport http` attempts HTTP first and falls back to SSE on v2.1.265+. | ✅ TRUE |
| `ws` | WebSocket remote servers; `--transport` flag does not accept `ws` — use `.mcp.json` or `add-json`. Header-only auth, no OAuth. | ⚠️ Draft omitted this transport |
| `sdk` | In-process servers, **reserved for SDK host applications** (Agent SDK, desktop app). Claude Code *skips and reports* a `type: "sdk"` entry in user/project configs. | ✅ TRUE — the draft was right; this sounded implausible but is confirmed |

## Command vectors for programmatic configuration

```bash
# Remote HTTP server (auth via --header)
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"

# Local stdio server; the double dash separates Claude's flags from the server command
claude mcp add <name> --scope <local|user|project> -- <command> [args...]

# Direct JSON blob ingestion (full schema in one payload)
claude mcp add-json <name> '<json>'
```

✅ TRUE — Ordering: all Claude-side options (`--transport`, `--env`, `--scope`, `--header`) precede the server name; everything after `--` is passed to the child process untouched. Without `--`, server flags get parsed as Claude Code options.

✅ TRUE — Management commands: `claude mcp list | get <name> | remove <name> [--scope <scope>]`; `/mcp` in-session for status, OAuth, enable/disable, reconnect.

Reserved names (`workspace`, `computer-use`, `claude-in-chrome`, `Claude Preview`, `Claude Browser`, etc.) are rejected by `claude mcp add` and skipped at load. ✅ TRUE (draft omitted)

## Importing from Claude Desktop

```bash
claude mcp add-from-claude-desktop [--scope user]
```

| Draft claim | Verdict |
|---|---|
| Works on macOS and WSL only | ✅ TRUE |
| Interactive checklist to select which servers to migrate | ✅ TRUE (interactive by default; single-server adds via `add`/`add-json` are the scripted alternative) |
| Imported servers written to the **global user configuration** | ❌ FALSE — default destination is **local scope** (per-project entry in `~/.claude.json`). Pass `--scope user` explicitly for all-projects availability. |
| Translation algorithm "automatically injects the stdio transport declaration" | ❓ UNVERIFIED as an implementation detail — but functionally moot: Claude Code reads typeless entries as stdio, so Desktop-style entries import without an explicit `type`. Treat the "algorithm" narrative as illustration, not spec. |
| Imported entries keep their name; duplicates get a numeric suffix | ✅ TRUE (e.g. `server_1`) |
| Desktop commands relying on shell `PATH`/relative paths may resolve differently under the CLI | ✅ TRUE — prefer absolute paths |

## Env variable expansion

✅ TRUE (draft omitted) — ${VAR} and ${VAR:-default} expansion is supported in `command`, `args`, `env`, `url`, and `headers`. Missing variables without a default produce a warning but the server still loads with the literal ${VAR} text. In remote `url`/`headers`, some credential variables read as empty instead. Claude Code sets `CLAUDE_PROJECT_DIR` in spawned stdio servers' environments.
