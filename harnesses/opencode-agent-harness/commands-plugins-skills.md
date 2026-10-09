# Commands, plugins, skills, and rule imports

## Custom commands

- **✅** Markdown files in `.opencode/commands/` or `~/.config/opencode/commands/`; filename → `/name`. Also via `command.<name>` in opencode.json (`template` required).
- Options: `description`, `agent` (subagent triggers subtask by default), `subtask` bool, `model`.
- **✅** Template syntax: `$ARGUMENTS`, positional `$1`/`$2`/…, `!\`cmd\`` injects shell output (runs at project root), `@file` includes file content.
- Built-ins: `/init`, `/undo`, `/redo`, `/share`, `/help`, `/connect`, `/models`.

## Rules / instruction imports

### AGENTS.md

- **✅** `/init` generates/improves `AGENTS.md` in place (build/test commands, structure, gotchas); recommended to commit it.
- Locations: project `AGENTS.md` (repo root, applies to dir + subdirs) and global `~/.config/opencode/AGENTS.md`.

### Claude Code compatibility (fallbacks, not a fork)

| OpenCode | Claude fallback | Disable env |
|---|---|---|
| Project `AGENTS.md` | project `CLAUDE.md` (only if no AGENTS.md) | `OPENCODE_DISABLE_CLAUDE_CODE` |
| `~/.config/opencode/AGENTS.md` | `~/.claude/CLAUDE.md` | `OPENCODE_DISABLE_CLAUDE_CODE_PROMPT` |
| skills dirs | `~/.claude/skills/` | `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS` |

- **✅** First match wins per category — `AGENTS.md` beats `CLAUDE.md` rather than merging.
- **✅** `instructions: ["*.md", "https://..."]` in config adds extra rule files (globs + remote URLs, 5s fetch timeout), combined with AGENTS.md — the "import configuration files" surface for rules.
- ⚠️ opencode does **not** auto-parse `@file` references inside AGENTS.md; use `instructions` globs or teach lazy-loading via the AGENTS.md text itself.

## Plugins

### Locations

- `.opencode/plugins/*.js|ts` (project) and `~/.config/opencode/plugins/` (global), auto-loaded at startup.
- npm packages via `plugin: ["name", "@scope/name"]` — installed by Bun at startup, cached in `~/.cache/opencode/node_modules/`.
- Local deps: `package.json` in the config dir → `bun install` at startup.

### Load order

1. Global config plugins
2. Project config plugins
3. `~/.config/opencode/plugins/`
4. `.opencode/plugins/`

All hooks run in sequence; same-name+version npm dupes load once.

### Plugin API

- **✅** `export const MyPlugin = async ({ project, client, $, directory, worktree }) => ({...hooks})`; `$` is Bun shell, `client` is the OpenCode SDK client. TypeScript types: `import type { Plugin } from "@opencode-ai/plugin"`.
- Return object may implement:
  - **`event` handler** — subscribes to the event bus
  - **hook keys** like `tool.execute.before`/`tool.execute.after` (mutate `output.args`)
  - **`tool` map** for custom tools
  - compaction/custom-context hooks

### Events (from docs)

- Command: `command.executed`
- File: `file.edited`, `file.watcher.updated`
- Install: `installation.updated`
- LSP: `lsp.client.diagnostics`, `lsp.updated`
- Message: `message.part.removed`, `message.part.updated`, `message.removed`, `message.updated`
- Permission: `permission.asked`, `permission.replied`
- Server: `server.connected`
- Session: `session.created|compacted|deleted|diff|error|idle|status|updated`
- Todo: `todo.updated`; Shell: `shell.env`
- Tool: `tool.execute.before`, `tool.execute.after`
- TUI: `tui.prompt.append`, `tui.command.execute`, `tui.toast.show`

## Skills

- **✅** Skill dirs: `.opencode/skills/` (project), `~/.config/opencode/skills/` (global), `~/.claude/skills/` (compat fallback, disable via `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS`).
- **✅** Loading a skill is permission-gated by the `skill` permission key.
