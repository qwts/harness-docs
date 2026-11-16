# MCP, connectors, skills, plugins, and hooks

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site main)

## MCP servers

“MCP servers extend the Vibe Code CLI with **external tools** through the Model Context Protocol … You add them with the `vibe mcp add` command, the `/mcp add` slash command, or directly in `config.toml` under `mcp_servers`.” “MCP servers are **user-configured**. For Mistral-managed integrations, see Connectors.” Admin gate: “Your administrator can block MCP servers with `enable_mcp = false` … Vibe refuses to add, remove, enable, or sign in to MCP servers, and loads no MCP tools.”

Transports: `http`, `streamable-http`, `stdio`. Examples: OAuth servers via `vibe mcp add linear --url https://mcp.linear.app/mcp` (“Vibe opens your browser to sign in”; `--no-login`, repeatable `--scope`), API-key servers via `--api-key-env MISTRAL_API_KEY` or `--header`, and stdio via `--command uvx --arg mcp-server-fetch` (repeat `--arg`/`--env`). “Remove a server with `vibe mcp remove <name>`. Removing an OAuth server also deletes its stored tokens.” Config form: `[[mcp_servers]]` tables with `[mcp_servers.auth]` (`type = "static"` …). `/mcp` (alias `/connectors`) “Browse MCP servers and connectors. Pass a name to list its tools. Subcommands: `add`, `status`, `login`, `logout`.”

**Tool naming for permissions**: “MCP tools are addressed as `{server_name}_{tool_name}`” — e.g. `[tools.fetch_server_get] permission = "always"`.

## Connectors

“Connectors give the Vibe Code CLI access to **supported first-party and third-party services** … Unlike MCP servers, which you wire up yourself, Connectors are **Mistral-managed integrations**.” Difference table: configured by — “Mistral, as integrated services” vs “You, with `vibe mcp add` or in `config.toml`”; auth model — “Account or service login” vs “OAuth sign-in, headers, environment variables, or API keys”; discoverability — “Curated, listed in the CLI” vs “Anything that speaks MCP”. “Connectors are detected and configured automatically”; `/connectors` lists and toggles them. “`enable_connectors = false`” blocks connectors “including the built-in web search.”

## Skills

“Skills are **reusable instruction sets** that extend Vibe Code with new workflows and custom slash commands. Vibe follows the Agent Skills specification, so skills you write here are portable to other agents that adopt the same spec.” A skill is “a **directory containing a `SKILL.md` file**” with YAML frontmatter: `name`, `description` (“Up to 1,024 characters”), `user-invocable` (default true — “exposes the skill as a slash command”; false hides it from the picker), `disable-model-invocation` (“stop the agent from loading the skill on its own. You can still run it with `/skill-name`”), and `allowed-tools` — “Vibe reads this field but doesn't enforce it: a skill can call any tool the active agent allows.”

Locations, in order: custom `skill_paths`, project `./.vibe/skills/` or `./.agents/skills/` (“when the working directory is trusted”), user `~/.vibe/skills/` or `~/.agents/skills/`. “A running session picks up skills you add or edit in these locations. If a change doesn't show up, run `/reload`.” Filters: `enabled_skills` / `disabled_skills` with “exact names, glob patterns, and regex” — “If `enabled_skills` is non-empty, it acts as an allow-list”; `disabled_skills` is “applied when `enabled_skills` is empty” (unlike `disabled_tools`, “applied after `enabled_tools`”).

## Plugins (experimental)

“Plugin support is experimental and under active development. The plugin format … will change, and an update can break existing plugins.” “A plugin is a **directory that bundles extensions** for Vibe Code: skills, MCP servers, knowledge folders, subagents, and more.” Format: “Vibe reads plugins in the Agent Plugins 1.0 format. Every plugin has a `plugin.json` manifest. Vibe-specific components live in an `ai.mistral.vibe/` directory.” “Vibe ships one built-in plugin, `vibe`, which provides skills such as `/vibe:skill-creator` and `/vibe:worktree`.”

Locations and precedence: project `./.vibe/plugins/<plugin>/` (“only when trusted”; `--add-dir` directories “always”), user `~/.vibe/plugins/<plugin>/`, built-in. “When two plugins share a `name`, the project plugin wins over the user plugin, which wins over the built-in one. If two plugins with the same name sit at the same scope, Vibe loads neither.” Sessions pin a snapshot (“Pinned plugins”; `/reload-plugins` re-pins), with `PLUGIN_ROOT` and `PLUGIN_DATA` set for plugin hooks and MCP servers. “Hooks in plugins written for Claude Code or Kimi Code don't run: Vibe reports them as not supported.”

## Hooks

“Hooks wire arbitrary shell commands into the Vibe Code CLI's lifecycle to **gate, audit, or rewrite** agent behavior. … Declare them in a `hooks.toml` file and they take effect on the next run.” Locations: `./.vibe/hooks.toml` (project, “**trusted folders only**”) then `~/.vibe/hooks.toml` (user); “the **project entry wins**”; “Subagents inherit the parent's hook configuration, so policies apply transitively.” Plugin hooks (in `ai.mistral.vibe/hooks.toml`) are name-prefixed (“`release-tools:audit-bash`”) and run “for each event … in this order: `./.vibe/hooks.toml`, project plugins, `~/.vibe/hooks.toml`, then user plugins.”

Hook shape: `[[hooks]]` with `name`, `type` (`pre_tool` / `post_tool` / `post_agent`), `command` (payload on stdin; “On Windows, runs in Git Bash when available, otherwise in PowerShell”), `match` (tool/agent name, “fnmatch glob, or regex with the `re:` prefix. Case-insensitive”), `timeout` (default 60), `strict` (failures become denials/clears instead of warnings), `description`. Contract: exit `0` + empty stdout = passthrough; exit `0` + JSON = structured response with `decision` (`"allow"`/`"deny"`), `reason`, `system_message`, `hook_specific_output`; “Unknown JSON fields are tolerated at every level.”

## ACP and other IDEs

“Vibe implements the Agent Client Protocol (ACP) and is published in the ACP registry … This means you can run Vibe in ACP-compatible clients outside the CLI, the VS Code extension, and Vibe Code Web,” including “JetBrains IDEs such as IntelliJ IDEA, PyCharm, WebStorm, and GoLand” via JetBrains AI Assistant. Clients without a registry can “point the client at the `vibe-acp` command” (in Zed, under `agent_servers`). Warning: “**Don't use the Mistral Code Enterprise JetBrains plugin.** It's deprecated.” The VS Code extension itself “runs the same Vibe agent as the CLI through the Agent Client Protocol” and adds “editor context … active file, selection, diffs, and source control.”

## Official sources

- [MCP servers](https://docs.mistral.ai/vibe/code/cli/mcp-servers) · [Connectors](https://docs.mistral.ai/vibe/code/cli/connectors) · [Skills](https://docs.mistral.ai/vibe/code/cli/skills) · [Plugins](https://docs.mistral.ai/vibe/code/cli/plugins) · [Hooks](https://docs.mistral.ai/vibe/code/cli/hooks) · [Use Vibe in other IDEs](https://docs.mistral.ai/vibe/code/use-vibe-in-other-ides)
