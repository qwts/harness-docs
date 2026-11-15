# Command Code agent harness

**Verified:** 2026-11-15 against the official Command Code documentation at commandcode.ai/docs and the `CommandCodeAI` GitHub organization  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** `cmd` CLI (interactive and headless), settings and config files, permission modes and rules, hooks, mods, AGENTS.md memory, taste learning, skills, subagents, MCP, sessions and checkpoints, BYOK providers, desktop app, and privacy controls  
**Provenance:** Official Command Code sources unless explicitly marked otherwise

Command Code (package `command-code`, binary `cmd`; `cmdc` on Windows where `cmd` is the OS shell) is a terminal coding agent positioned as “the best coding agent for open models. With taste.” Its differentiating layer is taste — a continuously learned preference profile stored as local markdown — and a harness-engineering layer (tool-call repairs, read/shell tool efficiency) the docs present as what makes open models usable. Configuration lives under `~/.commandcode/`, not `~/.config/`.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, settings, and providers](configuration-settings-and-providers.md) | `~/.commandcode/` layout, project files, precedence, environment variables, BYOK providers, auth, attribution, model resolution |
| [Permissions, hooks, and mods](permissions-hooks-and-mods.md) | Five permission modes, rule syntax and union, `disableBypass`, headless block, the four hook events, mods as the extension layer |
| [Memory, taste, and sessions](memory-taste-and-sessions.md) | AGENTS.md tiers and imports, taste-1 and `npx taste`, session transcripts, checkpoints and rewind, scratchpad, compaction |
| [Agents, skills, and MCP](agents-skills-and-mcp.md) | Subagent definitions and isolation, skills and progressive disclosure, MCP scopes and precedence |
| [Surfaces, CLI, desktop, and privacy](surfaces-cli-desktop-and-privacy.md) | Headless mode, CLI flags, interactive keys, desktop app, install/updates, telemetry and local-only mode |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Configuration lives in `~/.config/command-code/`.”** The config root is `~/.commandcode/`; project files are `.commandcode/` in the repo and `.mcp.json` at the root.
- ❌ **“A project allow rule overrides a user-global deny.”** Rule lists union across scopes and “deny wins, then ask, then allow” — specificity does not reorder anything.
- ❌ **“`--permission-mode yolo` is enough for headless runs that write files.”** Headless mode blocks writes unless `--yolo`/`--dangerously-skip-permissions` is passed; the mode flag alone “does not lift that block.”
- ❌ **“taste-1 is an LLM that sends your code to the cloud.”** The docs state it “is not an LLM”; taste is local markdown, and taste learning “runs locally and stores preferences as structured rules — not code snippets.”
- ❌ **“Command Code reads `CLAUDE.md`.”** It “reads AGENTS.md, not CLAUDE.md”; `/import` migrates other agents’ files.
- ❌ **“PostToolUse hooks can undo a tool call.”** A PostToolUse `decision: "block"` is an advisory retry signal — “the tool already ran, so nothing is un-done.”
- ❌ **“Mods run sandboxed.”** The docs are explicit: “There is no sandbox. A mod is code that runs with your permissions.”
- ⚠️ **“Command Code is fully Claude Code-compatible.”** Permission-mode aliases, `CLAUDE_*` skill placeholders, and `.agents/skills/` paths are honored — but config root, memory file, and skill roots differ.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Config root | `~/.commandcode/` (JSON files; nothing created on launch) |
| User files | `config.json` (preferences), `settings.json` (hooks/permissions/MCP), `auth.json` (0600, don’t edit), `providers.json` (BYOK), `keybindings.json`, `mcp.json`, `cron/jobs.json` |
| Project files | `.commandcode/settings.json` (commit), `.commandcode/settings.local.json` (gitignore), `.mcp.json` (commit) |
| Settings precedence | project `settings.local.json` → project `settings.json` → user `settings.json` → `config.json` (legacy fields); maps deep-merge, scalars overwrite, permission lists **union** |
| Permission modes | `default`, `accept-edits`, `plan`, `yolo`, `dont-ask`; shift+tab cycles the first four |
| Rule order | “deny wins, then ask, then allow”; lists accumulate across scopes |
| Headless | `cmd -p "query"`; writes blocked without `--yolo`; `--max-turns` default 100, exit code 8 on cap |
| MCP scopes | local `~/.commandcode/projects/<slug>/mcp.json` > project `.mcp.json` > user `~/.commandcode/mcp.json` |
| MCP tool names | `mcp__<server>__<tool>` |
| Memory | `AGENTS.md` tiers (user → project → subdir), `@path` imports recursive to 5 levels; “reads AGENTS.md, not CLAUDE.md” |
| Taste storage | `.commandcode/taste/` (project), `~/.commandcode/taste/` (global), `commandcode.ai/username/taste` (remote, opt-in push/pull) |
| Sessions | `~/.commandcode/projects/<slug>/<id>.jsonl` — append-only tree; per-turn commit |
| Checkpoints | one per prompt; kept 30 days, 200 backups/session, 10 MB/file cap |
| Subagents | Markdown files in `.commandcode/agents/` (project) and `~/.commandcode/agents/` (personal); first name wins; cannot spawn subagents |
| Default attribution | `Co-authored-by: CommandCodeBot <noreply@commandcode.ai>`; `attribution.commit: ""` disables |
| BYOK | `~/.commandcode/providers.json` only (user-level); `/connect` or by hand; keys in `auth.json` |
| Local-only mode | `cmd --local-only`, `CMD_LOCAL_ONLY=1`, or `"localOnly": true` — no telemetry, no billing reads, no `/share` |

## Official sources

- [Docs index](https://commandcode.ai/docs)
- [Settings & Config](https://commandcode.ai/docs/settings) · [Permissions](https://commandcode.ai/docs/permissions) · [Hooks](https://commandcode.ai/docs/hooks) · [Mods](https://commandcode.ai/docs/mods)
- [Memory](https://commandcode.ai/docs/memory) · [Taste](https://commandcode.ai/docs/taste) · [Sessions & Checkpoints](https://commandcode.ai/docs/sessions)
- [Agents](https://commandcode.ai/docs/agents) · [Skills](https://commandcode.ai/docs/skills) · [MCP](https://commandcode.ai/docs/mcp)
- [Interactive Mode](https://commandcode.ai/docs/interactive-mode) · [CLI Reference](https://commandcode.ai/docs/reference/cli) · [Desktop App](https://commandcode.ai/docs/desktop)
- [BYOK Providers](https://commandcode.ai/docs/byok) · [Harness Engineering](https://commandcode.ai/docs/harness-engineering)
- [Security & Privacy](https://commandcode.ai/docs/resources/security)
- [GitHub organization](https://github.com/CommandCodeAI)
