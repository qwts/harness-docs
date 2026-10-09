# Import Architecture, Rules & AGENTS.md (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against `cli/extensibility/rules`, `cli/extensibility/index`, `cli/reference/configuration/read-config-from`, `onboard-devin/agents-md`, and `desktop/cascade/*` (docs.devin.ai).

## Provider architecture: `read_config_from`

Devin CLI's defining architectural choice: instead of requiring Devin-native config, it **imports from other harnesses' config files** by default. Each provider key toggles one importer in `config.json` (user or project):

| Provider key | Default | Imports |
|---|---|---|
| `agents_standard` | `true` | `AGENTS.md`, `AGENTS.local.md`, `AGENT.md`, `.windsurfrules` (always-on rules) |
| `cursor` | `true` | Rules + MCP servers (`.cursor/rules/*.md|*.mdc`, MCP config) |
| `windsurf` | `true` | Rules + skills + MCP servers (`.windsurf/rules/*.md`, global_rules.md) |
| `claude` | `true` | Rules, skills, **commands**, MCP servers — plus `.claude/settings.json` hooks and custom subagents from `.claude/` |
| `copilot` | `true` | Skills only (`.github/skills/`, Copilot global skill dirs) |
| `opencode` | `true` | MCP servers only (`opencode.json`) |
| `zed` | `true` | MCP servers only (`.zed/settings.json`) |

✅ TRUE — every provider defaults ON; set `false` to opt out. This is the "import configuration files" surface: a project wired for Claude Code/Cursor/Windsurf needs zero Devin-specific files to start.

⚠️ Scope nuance: `copilot`, `opencode`, `zed` import a **subset** (skills or MCP only) — they are not full config importers.

## Rules: locations and format

| Location | Scope | Notes |
|---|---|---|
| `AGENTS.md` at repo root (or any dir) | project | Always-on; see cap below |
| `.devin/rules/*.md` | project | One rule per file, `trigger` frontmatter |
| `.devin/global_rules.md` | project | Single always-on file |
| `~/.devin/rules/*.md`, `~/.devin/global_rules.md` | user-global | Apply to every project |
| `~/.config/devin/AGENTS.md` | user-global | Global rules beside user config |
| `.windsurf/rules/*.md`, `.windsurf/global_rules.md`, `.windsurfrules` | project/user | Legacy formats, still read |

✅ TRUE — **`.devin/` takes precedence over `.windsurf/` for the global file**: if both `global_rules.md` exist, only the `.devin` one loads. But *directory* rules from both `.devin/rules/` and `.windsurf/rules/` are loaded together.
✅ TRUE — there is **no `.devinrules`** single-file equivalent; the only single-file root format is legacy `.windsurfrules`.
✅ TRUE — project rules are discovered at the workspace root **and every directory between root and cwd**.

### Activation triggers (`trigger:` frontmatter)

| `trigger:` | When the rule reaches the agent |
|---|---|
| `always_on` | Full content in system prompt every message |
| `model_decision` | Only `description` advertised; agent reads full file when relevant |
| `glob` | Applied when files matching `globs` pattern are read/edited |
| `manual` | Never auto-loaded; activated by `@rule-name` mention |
| `agent` | Agent-invoked variant |

✅ TRUE — same frontmatter grammar as `.windsurf/rules/*.md` (and Cascade workspace rules use the same four-mode table).

## AGENTS.md specifics

- ✅ TRUE — Devin auto-includes up to **16 KiB (16,384 bytes) from the beginning** of each `AGENTS.md`. Longer files → truncation notice + on-demand read; content past the cap is never auto-loaded. Keep critical instructions at the top.
- ✅ TRUE — discovery is hierarchical: repo root and nested files along the path to cwd are all eligible.
- ⚠️ PARTIAL — "AGENTS.md is one file at the root": in Devin Desktop's rules engine, a **root** `AGENTS.md`/`agents.md` is treated `always_on`, while a **subdirectory** file becomes an implicit `glob` rule with pattern `<directory>/**` — location *is* the activation mode.

## Skills

- Format: `skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`, optional `subagent`, `agent`, `model`, `allowed-tools`, `permissions`). ✅ TRUE
- The `.agents` skill standard is supported, so third-party skill installers work. ✅ TRUE
- ⚠️ `subagent: true` runs the skill as a foreground subagent on the `subagent_general` profile — **`allowed-tools`/`permissions` frontmatter is ignored** there. Use `agent: <custom-profile>` with `allowed-tools` to actually constrain it.
- Installed plugin skills surface as `/<plugin>:<skill>` slash commands.

## Migration note

Cascade "memories" and "workflows" are **not** supported by the Devin Local/CLI harness; skills are the documented migration target, and Devin Desktop ships a **Devin: Open Cascade Migration Wizard** command-palette flow for it.
