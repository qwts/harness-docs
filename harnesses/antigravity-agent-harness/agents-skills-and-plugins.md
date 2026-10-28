# Agents, skills, and plugins

**Verified:** 2026-10-29 · **Primary surfaces:** Antigravity App and CLI

## Custom subagents

Workspace discovery:

```text
.agents/agents/<name>.md
.agents/agents/<name>/agent.md
```

Global discovery:

```text
~/.gemini/config/agents/<name>.md
~/.gemini/config/agents/<name>/agent.md
```

Plugins can also provide `agents/` templates. A custom agent is Markdown with YAML frontmatter. Required fields are `name` and `description`. Other documented fields include `tools`, `mainAgent`, `subagent`, `model`, `commandExecutionPolicy`, `mcpServers`, `skills`, and `plugins`.

A known documentation issue warns that an unmapped or misspelled tool name in `tools` can cause the subagent process to hang. Validate exact tool names rather than assuming invalid entries are harmless.

### Runtime isolation

The parent invokes subagents with `invoke_subagent`. A child starts with clean conversation context and runs concurrently. Workspace options are:

- `inherit` — inherit the parent's workspace;
- `branch` — create an isolated Git worktree;
- `share` — share directory storage.

Subagents inherit parent safety scopes, including allowed command prefixes, file directory scopes, and sandbox settings. Approval requests bubble to the main UI or subagent panel. Parents retain access to child workspaces, including isolated worktrees.

Built-in `research`, `browser`, and `self` roles have different purposes. `/boost` and `/teamwork-preview` are higher-level multi-agent orchestrators, not merely alternate filenames for one custom subagent.

## Skills

A skill is a directory containing `SKILL.md` with required `name` and `description` frontmatter, plus optional scripts, examples, and resources. The agent sees names/descriptions and reads full instructions when relevant.

App locations:

```text
Workspace: .agents/skills/<skill>/
Global:    ~/.gemini/config/skills/<skill>/
```

The CLI migration guide documents workspace `.agents/skills/` and global `~/.gemini/antigravity-cli/skills/`; legacy Gemini CLI paths are not interchangeable with current CLI paths. Skills can be invoked autonomously or manually as `/<skill-name>` on supported surfaces.

## Plugins

A plugin is a directory with root `plugin.json` and optional components:

```text
plugin.json
mcp_config.json
hooks.json
skills/<skill>/SKILL.md
agents/<agent>.md
rules/<rule>.md
```

Installation locations:

```text
Workspace: .agents/plugins/<plugin>/
Global:    ~/.gemini/config/plugins/<plugin>/
```

`name` is required when managing a plugin through CLI commands. In App and IDE it is optional and defaults to the folder name. The manifest can reference the official schema for editor validation.

Plugin MCP configuration supports `${PLUGIN_ROOT}` for the install directory and `${PLUGIN_DATA}` for persistent data outside the workspace. This differs from Cursor's documented plugin-variable behavior; portability must be checked per client.

The App marketplace and CLI `/plugin` / `agy plugin` management surfaces can install and manage bundles. A plugin is a packaging mechanism, not a guarantee that every component behaves identically across App, CLI, and IDE.

## Sources

- [Custom subagents](https://antigravity.google/docs/subagents/)
- [Agent skills](https://antigravity.google/docs/skills/)
- [Plugins](https://antigravity.google/docs/plugins/)
- [Gemini CLI migration](https://antigravity.google/docs/cli/gcli-migration/)
