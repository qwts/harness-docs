# Instructions, skills, and plugins

**Verified:** 2026-11-15

## AGENTS.md discovery

Amp’s instruction files are `AGENTS.md`, discovered in layers:

- Files in the current working directory (or editor workspace roots) and parent directories up to `$HOME` are always included; subtree files are included only when the agent reads a file in that subtree.
- System-wide files (`/etc/ampcode/AGENTS.md`, `/Library/Application Support/ampcode/AGENTS.md`, `%ProgramData%\ampcode\AGENTS.md`) and personal global files (`$HOME/.config/amp/AGENTS.md`, `$HOME/.config/AGENTS.md`) are always included when present.
- If a directory has no `AGENTS.md` but has `AGENT.md` (singular) or `CLAUDE.md`, that fallback is included instead.

The web app adds two more layers: a personal Global AGENTS.md (Settings → Instructions) and a workspace-wide one set by admins. “Amp includes workspace guidance before personal and repository guidance,” and workspace guidance “does not apply to subagents, Puck, or raw custom agents.” Both global layers also apply to threads that run the Claude Code agent.

`@`-mentions reference other guidance files with relative paths (resolved against the mentioning file) or `@~/path`; absolute paths are not supported, mentions inside code blocks are ignored, and globs like `@doc/*.md` work. Front-matter `globs:` make a mentioned file apply only after Amp reads a matching file. `AMP_IGNORE_GUIDANCE_FILES` (colon-delimited globs, `'*'` for all) excludes files from guidance — though the agent can still read them explicitly with a tool.

## Skills

A skill is a directory with a `SKILL.md` (YAML frontmatter `name`/`description` plus markdown, with scripts and reference files alongside). Amp sees each skill’s name and description and “loads the rest of SKILL.md only when the skill is invoked.”

Search order — Amp “uses the first skill with a given frontmatter name”:

1. `~/.config/agents/skills/`
2. `~/.agents/skills/`
3. `~/.config/amp/skills/`
4. `.agents/skills/` in the project and parent directories
5. `.claude/skills/` in those directories
6. `~/.claude/skills/`
7. `~/.claude/plugins/cache/`
8. Directories from `amp.skills.path`
9. Personal skills repository
10. Active workspace skills repository
11. Official Amp skills

Local skills mask repository skills, and a personal skill masks a workspace skill with the same name. Claude-compatible locations are searched by default; `amp.skills.disableClaudeCodeSkills` and `amp.skills.disableGlobalAgentsSkills` turn those classes off. Skills can bundle MCP servers via a sibling `mcp.json` or `mcpServers` frontmatter (frontmatter wins); skill servers are the lowest MCP precedence, get `AMP_SKILL_DIR` set, and hide their tools until the skill loads.

Personal and workspace skills live in separate git repositories — a push publishes the skill — managed with `amp skills repositories`, `amp clone user-skills`/`workspace-skills`, and `amp skill import`/`update`. Hosted repos cap at 200 skills (alphabetically first on overflow), 200 files per skill, 10 MiB per file, 25 MiB per skill and per repo, text files only; project and machine-local directories have no such limits.

## Plugins

Plugins are “TypeScript or JavaScript modules that add tools, commands, and event-driven behavior to Amp” — and “run code in your environment, so only load plugins you trust.” A plugin is a single file or a directory with `index.ts`/`index.js` exporting a default function that receives a `PluginAPI` object. Locations: project `.amp/plugins/`, system `$XDG_CONFIG_HOME/amp/plugins/` (or `~/.config/amp/plugins/`, `%USERPROFILE%\.config\amp\plugins\`), plus personal and workspace repositories managed in settings. Name collisions resolve in the order project → system → personal → workspace.

The interception points:

- `tool.call` fires before a tool runs and returns one of `allow`, `reject-and-continue`, `modify`, or `synthesize`.
- `tool.result` fires after a tool finishes and can replace the result.
- Lifecycle runs `session.start` → `agent.start` → `tool.call` → `tool.result` → `agent.end`; there is no `session.end`. `agent.start` may return hidden instructions appended to the user turn; `agent.end` may return `continue` to start another turn — chained continues stop after five by default (`maxContinuations` raises it).

Plugins also register tools (`amp.registerTool`), commands, custom agents (`amp.createAgent`, `amp.registerAgentMode`), bundle skills, and call AI helpers (`amp.ai.ask`/`amp.ai.decide`). They install via `amp plugins add <url>` (`--target workspace` for project-local), live in git-backed personal/workspace repos with hosted limits of 50 plugins, 500 files per directory plugin, 10 MiB per file, and 25 MiB per plugin and repo, and reload via the `plugins: reload` palette command. Spaces and Puck have no executor, so plugins do not load in them.

## Official sources

- [AGENTS.md](https://ampcode.com/docs/customize/agents-md)
- [Skills](https://ampcode.com/docs/customize/skills)
- [Plugins](https://ampcode.com/docs/customize/plugins)
- [Plugin API](https://ampcode.com/docs/plugin-api)
- [Global plugins and skills](https://ampcode.com/docs/customize/global-plugins-and-skills)
