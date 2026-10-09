# OpenCode Agent Harness (OpenCode TUI + OpenCode Desktop + server)

> Verified 2026-10-09 against [opencode.ai/docs](https://opencode.ai/docs/) and the [anomalyco/opencode](https://github.com/anomalyco/opencode) repo. This entry covers the OpenCode agent harness across its surfaces: the terminal TUI (`opencode`), the desktop app (beta), and the headless/`opencode serve` API surface. The config/plugin machinery below is documented for the CLI/TUI/server surfaces; Desktop parity is not yet established (see cli-and-desktop-surfaces.md).

**Status**: complete, verified. **Provenance**: written directly against official docs — there was no source draft, so `corrections.md` is a common-misinformation audit rather than a diff log.

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
| [config-and-settings.md](config-and-settings.md) | opencode.json locations, the 8-source merge order, `.well-known/opencode` remote org config, managed/MDM settings, env vars, tui.json |
| [agents-and-permissions.md](agents-and-permissions.md) | primary agents vs subagents, built-ins, per-agent config (JSON + markdown), the permission model (`allow`/`ask`/`deny`, granular rules, `--auto`, `external_directory`, `doom_loop`) |
| [providers-and-mcp.md](providers-and-mcp.md) | provider architecture (AI SDK + Models.dev, 75+ providers), credentials store, OpenCode Zen/Go, MCP local + remote servers, OAuth, per-agent MCP scoping |
| [commands-plugins-skills.md](commands-plugins-skills.md) | custom commands, plugins (dirs + npm, hook events, custom tools), skills, rule/instruction imports incl. Claude Code compatibility |
| [cli-and-desktop-surfaces.md](cli-and-desktop-surfaces.md) | install paths, TUI commands/keybinds, `opencode run`/`serve`/non-interactive, desktop app (beta), config surfaces per product |
| [corrections.md](corrections.md) | Audit of common misinformation: config-merge myths, stale `tools` config, path conventions, agent/type confusion |

## Top corrections (read first if skimming)

1. **❌ Config files do not replace each other — they merge.** Remote → global → `OPENCODE_CONFIG` → project → `.opencode` dirs → `OPENCODE_CONFIG_CONTENT` → managed dirs → MDM profile, later overriding earlier only on conflicting keys.
2. **⚠️ The project config is `opencode.json`, not `.opencode/config.json`.** `.opencode/` is a directory of subdirs (`agents/`, `commands/`, `plugins/`, `skills/`); `opencode.json` sits at the repo root.
3. **❌ `tools` boolean config is not the current permission system.** Since v1.1.1 it is deprecated and merged into `permission` (allow/ask/deny per tool with glob rules). Old `tools` entries still parse.
4. **⚠️ Both `tools` and `permission` exist per-agent.** Agent `permission` merges over global; agent rules win.
5. **⚠️ `.opencode` uses plural subdirs, but singular still works.** `agents/`, `commands/`, `plugins/`, `skills/` are canonical; `agent/`, `command/` etc. are kept for backwards compatibility.
6. **❌ "opencode is Claude Code / Windsurf with another name" is false.** It is the SST/anomalyco open-source agent (formerly `opencode-ai`). Claude Code compatibility is only a file-convention fallback (`CLAUDE.md`, `~/.claude/`), not a fork.
7. **⚠️ Rules don't merge across locations — first match wins per category.** Project `AGENTS.md` beats `CLAUDE.md`; global `~/.config/opencode/AGENTS.md` beats `~/.claude/CLAUDE.md`. Extra files need the `instructions` config key.
8. **⚠️ MCP OAuth needs no client config in the common case.** OpenCode auto-detects 401s and runs Dynamic Client Registration; `oauth: false` disables it for API-key servers.
9. **⚠️ `opencode run` and `opencode serve` are the headless surfaces**, not a separate "server build" — `server.port` in config and `opencode serve` expose the same harness as an API.
10. **⚠️ The desktop app is in beta** and ships `opencode-desktop-*` artifacts (brew/scoop/GitHub releases); it is not the same binary path as the TUI install scripts.

## Quick reference

| Surface | Path / handle |
|---|---|
| Global config | `~/.config/opencode/opencode.json` (`.jsonc` ok) |
| Project config | `<repo>/opencode.json` (repo root, upward search to git root) |
| Remote org config | `<provider>/.well-known/opencode` (fetched on auth) |
| Managed config | `/etc/opencode/` (Linux), `/Library/Application Support/opencode/` (macOS), `%ProgramData%\opencode` (Windows) |
| TUI config | `~/.config/opencode/tui.json` (+ project `tui.json`) |
| Agents | `.opencode/agents/*.md`, `~/.config/opencode/agents/` |
| Commands | `.opencode/commands/*.md`, `~/.config/opencode/commands/` |
| Plugins | `.opencode/plugins/*.ts|js`, `~/.config/opencode/plugins/`, npm via `plugin: []` |
| Skills | `.opencode/skills/`, `~/.config/opencode/skills/`, `.claude/skills/` + `~/.claude/skills/` (compat), `.agents/skills/` + `~/.agents/skills/` |
| Rules | `AGENTS.md`, `CLAUDE.md` fallback, `~/.config/opencode/AGENTS.md`, `instructions: []` key |
| Credentials | `~/.local/share/opencode/auth.json`; MCP OAuth tokens `~/.local/share/opencode/mcp-auth.json` |
| Env overrides | `OPENCODE_CONFIG`, `OPENCODE_CONFIG_DIR`, `OPENCODE_CONFIG_CONTENT`, `OPENCODE_DISABLE_CLAUDE_CODE*` |

## Sources

- https://opencode.ai/docs/config/ — config locations, precedence, managed settings, env vars
- https://opencode.ai/docs/agents/ — agent types, built-ins, JSON/markdown config
- https://opencode.ai/docs/permissions/ — allow/ask/deny, granular rules, `--auto`, safety guards
- https://opencode.ai/docs/providers/ — AI SDK + Models.dev, `/connect`, auth.json, Zen/Go
- https://opencode.ai/docs/mcp-servers/ — local/remote MCP, OAuth, tool gating
- https://opencode.ai/docs/commands/ — command files, `$ARGUMENTS`, `!` shell, `@` file refs
- https://opencode.ai/docs/plugins/ — plugin API, events, load order, npm install
- https://opencode.ai/docs/rules/ — AGENTS.md, Claude Code compat, `instructions`, precedence
- https://opencode.ai/download + github.com/anomalyco/opencode — desktop app (beta) artifacts
