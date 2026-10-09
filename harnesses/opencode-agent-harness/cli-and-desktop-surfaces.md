# CLI, TUI, and desktop surfaces

## Install

- **✅** Canonical: `curl -fsSL https://opencode.ai/install | bash`. Also npm/bun/pnpm, Homebrew (`opencode`), and GitHub releases. Repo: `github.com/anomalyco/opencode` (MIT; formerly `opencode-ai`).
- TUI config: `~/.config/opencode/tui.json` (+ project `tui.json`); everything else shares opencode.json.

## Surfaces

| Surface | What it is |
|---|---|
| `opencode` (TUI) | Interactive terminal UI — the primary surface; Tab cycles primary agents, `/` opens command palette |
| `opencode run` | Headless one-shot mode (`opencode run "task"`, `--auto` for auto-approve) |
| `opencode serve` | API server (`server.port` config); the same harness exposed over HTTP |
| `opencode mcp *` | MCP management/auth subcommands |
| Desktop app (BETA) | `opencode-desktop-*` builds: macOS arm64/x64 dmg, Windows x64 exe, Linux deb/rpm/AppImage; `brew install --cask opencode-desktop`, scoop `extras/opencode-desktop` |
| Web/IDE | opencode.ai also ships a web app + IDE extension per intro docs ❓ (details not deep-verified) |

## TUI essentials

- `/connect` provider auth → `~/.local/share/opencode/auth.json`
- `/models` model picker; `/init` AGENTS.md generation; `/undo`/`/redo`; `/share` session sharing
- Command palette toggles auto-approve permissions (shows muted `auto` indicator)
- `@` mentions invoke subagents; `@file` attaches files in prompts
- Session navigation keys: `session_child_first`, `session_child_cycle`(+`_reverse`), `session_parent` — navigate into/out of subagent child sessions

## Features worth noting for agent consumption

- **✅** Multi-file rule layer: AGENTS.md + `instructions` globs + CLAUDE.md fallback
- **✅** Granular permission engine incl. `external_directory` and `doom_loop` safety guards
- **✅** MCP local + remote with auto-OAuth/DCR and token store
- **✅** Plugins = real Bun JS/TS modules with event bus + tool hooks + custom tools
- **✅** Built-in subagents (general/explore/scout) + markdown-defined custom agents
- **✅** Remote org config via `.well-known/opencode`; managed/MDM layer for enterprises
- **✅** `autoupdate` in config; JSONC support; `{env:}`/`{file:}` interpolation in config values

## What is NOT verified here

- Desktop app internal agent-harness differences vs the TUI (beta; docs do not yet document a separate config surface — treated as the same harness pending evidence) ❓
- `opencode web`/IDE extension details ❓
