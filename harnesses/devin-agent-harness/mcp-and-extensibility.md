# MCP, Plugins, Subagents & Hooks (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against `cli/extensibility/mcp/*`, `cli/extensibility/plugins/*`, `cli/extensibility/hooks/*`, `cli/subagents`, `cli/reference/commands`, `desktop/cascade/mcp` (docs.devin.ai).

## MCP configuration files

✅ TRUE — since **v3000.3 (Local 3.6)**, MCP servers live in dedicated files beside each config.json; `mcpServers` keys in main config files are **auto-migrated** to them on startup:

| Scope | File | Share |
|---|---|---|
| User | `~/.config/devin/mcp_config.json` (`%APPDATA%\devin\mcp_config.json` on Windows) | All your projects |
| Project | `.devin/mcp_config.json` | Committed, team-shared |
| Project-local | `.devin/mcp_config.local.json` | Gitignored, personal keys |

Schema — `mcpServers` map; stdio and remote entries:

```json
{
  "mcpServers": {
    "local-server":  { "command": "npx", "args": ["-y", "@co/mcp"], "env": {"KEY": "v"} },
    "remote-server": { "url": "https://mcp.example.com/mcp", "transport": "http" }
  }
}
```

## `devin mcp` commands

| Command | Notes |
|---|---|
| `devin mcp add <name>` | `-t stdio|http` (inferred: URL → http, trailing args → stdio); `-s local|project|user` (**default `local`**); `-e KEY=VALUE`; `-H 'K: V'`; `--scopes`; `--oauth-resource`; `--url` or positional URL; `-- <cmd>` for stdio |
| `list` / `get` / `remove` | `remove` takes `-s` (default `local`) |
| `login <name>` / `logout <name>` | OAuth flow; `--scopes`, `--oauth-resource` (RFC 8707 resource override; empty string omits it) |
| `enable` / `disable` | Soft-disable without deleting config |

✅ TRUE — HTTP transport tries Streamable HTTP first, **falls back to legacy SSE on non-auth 4xx** (404/405) per MCP spec; 401/insufficient-scope 403 with `WWW-Authenticate` surfaces as an auth prompt. `"transport": "sse"` can be set explicitly.
✅ TRUE — MCP tool gating uses permission patterns `mcp__server__tool`, `mcp__server__*`, `mcp__*` inside `permissions` lists.

## Plugins

✅ TRUE — the **plugin is the unit of installation** (skills can't be installed individually); compatible with Agent Plugins 1.0.0 spec. Layout:

```
my-plugin/
├── .devin-plugin/plugin.json   # manifest
├── AGENTS.md                   # always-on rule
├── rules/                      # triggered rules (same frontmatter)
├── agents/<name>.md            # custom subagents
├── hooks.json                  # lifecycle hooks
├── .mcp.json                   # MCP servers
└── skills/<name>/SKILL.md
```

- Install: `devin plugins install <owner/repo|git-url|path[#subdir]>`, `list`, `info`, `update`, `remove`, `prune` (GC unreferenced content). `-y` skips the trust prompt; `--force` removes despite governance requirements.
- Plugin skills → `/<plugin>:<skill>`; plugin subagents → `<plugin>:<name>` (**local surfaces only** — CLI and Desktop, not cloud sessions).
- ⚠️ Plugin `hooks.json` is **best-effort, fail-open** — don't rely on it for guardrails.
- Plugin MCP: `.mcp.json` or `mcpServers` field (string path / path list / `{paths, exclusive}` / inline map); literal secrets are stripped, `${NAME}` references allowed; **OAuth client secrets are rejected at activation** (client ID + scopes OK).
- `requiredPlugins` chains install recursively; one repo can host multiple plugins as `#subdirs`.

## Custom subagents

- Profiles: `.devin/agents/<name>.md` or `.devin/agents/<name>/AGENT.md` at project root (also `.agents/agents/`; global: `~/.config/devin/agents/`). Within a plugin source the unqualified `agents/` layout applies. Frontmatter controls system prompt, `allowed-tools`, `model`. `devin doctor` validates profiles.
- **Foreground** subagents prompt for tool approvals normally (prompt names the subagent). **Background** subagents inherit already-granted permissions and **auto-deny anything new** — they can't prompt.
- Skill `subagent: true` uses `subagent_general` (all tools). Tool names matched by hooks/permissions: `read`, `write`, `edit`, `apply_patch`, `notebook_*`, `grep`, `glob`, `exec`, `get_output`, `write_to_process`, `kill_shell`, `webfetch`, `todo_write`, `exit_plan_mode`, `skill`, `run_subagent`, `read_subagent`.

## Lifecycle hooks

✅ TRUE — events: `PreToolUse`, `PostToolUse`, `PermissionRequest`, `UserPromptSubmit`, `Stop`, `PostCompaction`, `SessionStart`, `SessionEnd`.

Project-level sources:
- `.devin/hooks.v1.json` (recommended standalone)
- `"hooks"` key in `.devin/config.json` / `.devin/config.local.json`
- `.claude/settings.json` / `.claude/settings.local.json` `"hooks"` key (Claude Code format compatibility)

System-level (org-wide, Devin Desktop): `/Library/Application Support/Devin/hooks.json`, `/etc/devin/hooks.json`, `C:\ProgramData\Devin\hooks.json` — falls back to legacy `Windsurf` paths if absent; takes precedence over user/workspace hooks; also distributable via cloud dashboard.

Payload shape follows the Claude Code convention — `{matcher, hooks:[{type:"command", command, timeout}]}`; stdin carries `tool_name` + event data; `hookSpecificOutput.additionalContext` on stdout injects context (e.g., SessionStart).
