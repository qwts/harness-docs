# Skills, MCP, and plugins

**Verified:** 2026-11-15

## Agent Skills

Skills follow the open Agent Skills format: a directory containing a `SKILL.md` with YAML frontmatter. Required fields: `name` — “Max 64 characters. Lowercase letters, numbers, and hyphens only. Must not start or end with a hyphen” — and `description` — “Max 1024 characters. Describes what the skill does and when to use it.”

Locations and precedence:

| Location | Scope | Notes |
|---|---|---|
| `~/.agents/skills/` + `.agents/skills/` | user + project | “Open agent standard, loaded by default” |
| `~/.kilo/skills/` | user | Kilo’s own user-level directory |
| `.kilo/skills/` | project | project skills take precedence over global on name conflicts |
| `~/.claude/skills/` + `.claude/skills/` | user + project | “Claude Code compatibility, loaded when Claude Code Compatibility is enabled” |
| `skills.paths` / `skills.urls` | config | extra paths (absolute, `~/`, or project-relative) and remote URLs serving an `index.json` manifest |

Kilo “discovers this user-level directory by default, without a `skills.paths` entry or a plugin to register the skills.” On conflicts, “project-level skills (`.kilo/skills/`) take precedence over global skills (`~/.kilo/skills/`).” There is no built-in marketplace UI yet: “skills from the Kilo Marketplace repository can be manually downloaded into your `.kilo/skills/` directory or loaded via `skills.urls` in config.”

**Embedded shell commands.** A skill body can embed `` !`command` `` placeholders — “When the agent loads the skill, each command runs and its standard output replaces the placeholder before the skill content reaches the model.” Safeguards, quoted:

- “Trusted skills only — commands execute only in skills from trusted locations: global skills (such as `~/.kilo/skills/`, `~/.agents/skills/`, and `~/.claude/skills/`), skills built into Kilo Code, and absolute skill paths declared in global config. Project skills (`.kilo/skills/` in a repository) and skills fetched from remote URLs never execute commands.”
- “Approval required — when the agent loads a trusted skill containing commands, every command in the file is listed in a single permission prompt before anything runs. Approving runs all of them; rejecting aborts the skill load. This prompt appears even when bash commands are otherwise auto-approved, and a deny rule on any command still blocks it.”
- “Kill switch — set the `KILO_DISABLE_SKILL_SHELL` environment variable to disable embedded command execution entirely.”

Commands run in the project directory with a per-command timeout; output is truncated before inlining, placeholders inside fenced code blocks never execute, and “command output is never re-scanned for further placeholders.”

## MCP servers

MCP servers are declared under the `mcp` key in `kilo.jsonc` (global `~/.config/kilo/` or project root/`.kilo/`). Each entry names a server and takes `type` plus connection fields:

| Type | Fields | Notes |
|---|---|---|
| `local` | `command` (array), `environment`, `enabled`, `timeout` | STDIO child process. CLI docs: tool-fetch “Default: 5000” ms. Extension docs: “The default is 10 seconds for local servers and 15 seconds for remote servers.” |
| `remote` | `url`, `headers`, `enabled`, `timeout`, `oauth` | “Kilo Code tries StreamableHTTP first, then falls back to SSE automatically.” “Remote servers support OAuth 2.0 authentication. If the server supports it, Kilo Code will automatically start the OAuth flow when you connect. You can also disable OAuth with `"oauth": false`.” |

The bare SSE transport is deprecated “as of MCP specification version 2025-03-26.” `{env:VAR}` substitution works in server configs. At runtime, tool calls go through the Permission Dock — “Click Approve Always to save an allow rule to your config so future calls to that tool are auto-approved” — and permission keys are `{server}_{tool}` with glob support. Servers can be disabled with `enabled: false` without deletion, and curated servers can be installed from the Kilo Marketplace. None ship pre-installed.

## Plugins

Plugins are TypeScript/JavaScript modules registered via the `plugin` array in `kilo.jsonc` (npm name, `name@version`, `[name, options]`, or a local `./path`), or by dropping files into a `plugin/`/`plugins/` folder: global `~/.config/kilo/plugin/`, project `.kilo/plugin/` (legacy `.kilocode/plugin/` also works). npm plugins “are installed automatically at startup using Bun. Packages and their dependencies are cached under `packages/` in the current CLI XDG cache directory” — `~/.cache/opencode/packages/` by default — and “Kilo installs packages with lifecycle scripts such as `install` and `postinstall` blocked.”

Load order: built-in plugins → global config array → global plugin directory → project config array → project plugin directory; “Hooks from multiple plugins run sequentially in load order.” `KILO_PURE=1` makes Kilo “skip all external plugins — only built-in plugins will load. Useful for reproducible CI runs or debugging.”

Plugin hooks (plugin exports object): `config`, `event`, `tool`, `tool.execute.before`/`tool.execute.after`, `tool.definition`, `chat.message`, `chat.params`, `chat.headers`, `permission.ask`, `command.execute.before`, `shell.env`, `auth`, `provider`, plus experimental hooks (`experimental.chat.messages.transform`, `experimental.chat.system.transform`, `experimental.session.compacting`, `experimental.compaction.autocontinue`, `experimental.text.complete`). Events include `session.*`, `message.*`, `tool.execute.*`, `permission.asked`/`permission.replied`, `file.edited`, `file.watcher.updated`, `shell.env`, `command.executed`, `lsp.*`, `todo.updated`, and `server.connected`.

## Official sources

- [Skills](https://kilo.ai/docs/customize/skills) · [Using MCP in Kilo Code](https://kilo.ai/docs/automate/mcp/using-in-kilo-code) · [Using MCP in CLI](https://kilo.ai/docs/automate/mcp/using-in-cli) · [MCP permissions](https://kilo.ai/docs/automate/mcp/mcp-permissions) · [Server transports](https://kilo.ai/docs/automate/mcp/server-transports) · [Plugins](https://kilo.ai/docs/automate/extending/plugins) · [MCP overview](https://kilo.ai/docs/automate/mcp/overview)
