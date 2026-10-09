# Config and settings

## Config file format

- **✅** OpenCode config is JSON or JSONC. Recommended header: `"$schema": "https://opencode.ai/config.json"`.
- **✅** Files are **merged, not replaced** — later sources override earlier ones only on conflicting keys; non-conflicting settings from all sources are preserved.

## The 8-source precedence order

Loaded in this order (later overrides earlier):

| # | Source | Role |
|---|---|---|
| 1 | Remote config — `https://<provider>/.well-known/opencode` | Org defaults, fetched automatically on provider auth |
| 2 | Global config — `~/.config/opencode/opencode.json` | User preferences |
| 3 | Custom config — `OPENCODE_CONFIG` env var | Custom overrides file |
| 4 | Project config — `opencode.json` in project root | Project settings (found by walking up to nearest git dir) |
| 5 | `.opencode` directories | agents/commands/plugins/skills dirs |
| 6 | Inline config — `OPENCODE_CONFIG_CONTENT` env var | Runtime overrides (inline JSON) |
| 7 | Managed config dirs — `/etc/opencode/` (Linux), `/Library/Application Support/opencode/` (macOS), `%ProgramData%\opencode` (Windows) | Admin-controlled, root/admin-only to write |
| 8 | macOS managed preferences (`.mobileconfig` via MDM) | Highest priority, not user-overridable |

- **✅** So: project beats global, global beats remote org defaults, managed beats everything.
- **✅** `OPENCODE_CONFIG_DIR` points to an extra config *directory* searched for agents/commands/modes/plugins like `.opencode/`, loaded after global config and `.opencode` dirs.
- **✅** `.opencode` and `~/.config/opencode` use **plural** subdirs — `agents/`, `commands/`, `modes/`, `plugins/`, `skills/`, `tools/`, `themes/`; singular names are a backwards-compat alias.

## Remote org config

- **✅** Any provider serving `/.well-known/opencode` can ship org defaults (e.g. MCP servers `enabled: false` by default that users opt into locally).

## TUI settings

- **✅** `~/.config/opencode/tui.json` for global TUI prefs; a `tui.json` beside project `opencode.json` for project TUI settings.

## Key config keys (schema-verified)

| Key | Type | Notes |
|---|---|---|
| `model` | `provider/model-id` string | e.g. `anthropic/claude-sonnet-4-5`, `opencode/gpt-5.1-codex` for Zen |
| `autoupdate` | bool or `"notify"` | `"notify"` checks without installing (non-pkg-manager installs) |
| `server.port` | number | API server port (`opencode serve`) |
| `permission` | string/object | see agents-and-permissions.md |
| `tools` | object | ⚠️ deprecated since v1.1.1 → merged into `permission` |
| `agent` | object | named agent configs (`mode`, `model`, `prompt`, `permission`, `temperature`, `steps`, `disable`, `description`) |
| `command` | object | named custom commands (`template` required, `description`, `agent`, `subtask`, `model`) |
| `mcp` | object | named MCP servers (see providers-and-mcp.md) |
| `provider` | object | per-provider options: `options.baseURL`, `blacklist`, `whitelist` |
| `plugin` | array of npm names | auto-installed via Bun into `~/.cache/opencode/node_modules/` |
| `instructions` | array | extra rule files (globs ok, URLs fetched w/ 5s timeout), combined with AGENTS.md |
| ⚠️ `theme`, `keybinds` | legacy | deprecated here — live in `tui.json` (`theme` is a string there) |

## Env vars

| Var | Effect |
|---|---|
| `OPENCODE_CONFIG` | path to custom config file (slot 3) |
| `OPENCODE_CONFIG_DIR` | extra config directory (slot ~5.5) |
| `OPENCODE_CONFIG_CONTENT` | inline JSON config (slot 6) |
| `OPENCODE_DISABLE_CLAUDE_CODE` | kill all `.claude` fallbacks |
| `OPENCODE_DISABLE_CLAUDE_CODE_PROMPT` | kill only `~/.claude/CLAUDE.md` fallback |
| `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS` | kill only `~/.claude/skills` fallback |

## Managed settings

- **✅** File-based managed config dirs require admin/root to write; MDM `.mobileconfig` on macOS tops the stack. Used to enforce things users cannot override.

## Interpolation

- **✅** Config values support `{env:VAR}` (e.g. `"Authorization": "Bearer {env:MY_API_KEY}"`) and `{file:path}` (e.g. `prompt: "{file:./prompts/build.txt}"`, relative to the config file location).
