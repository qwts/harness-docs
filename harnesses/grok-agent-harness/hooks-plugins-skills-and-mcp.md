# Hooks, plugins, skills, and MCP

**Verified:** 2026-11-15

## Hooks: 14 events, only PreToolUse blocks

“A hook is a shell command or HTTP endpoint that Grok calls when a lifecycle event occurs.” Hooks are JSON files: “Personal hooks live in `~/.grok/hooks/*.json`; project hooks live in `<project>/.grok/hooks/*.json`.” Extra hook roots come from `~/.grok/hooks/` (“extra roots via `~/.grok/hooks-paths`”) and enabled plugins, and — compatibility — “Claude Code (`.claude/settings.json`) and Cursor (`.cursor/hooks.json`) hook files are read as well, including Cursor’s camelCase event names.”

| Event | Fires when |
|---|---|
| `SessionStart`, `SessionEnd` | A session starts or ends |
| `UserPromptSubmit` | You submit a prompt |
| `PreToolUse` | “A tool is about to run — the only blocking event” |
| `PostToolUse`, `PostToolUseFailure` | A tool completes or fails |
| `PermissionDenied` | “The permission system denies a tool call” |
| `Stop`, `StopFailure` | “A turn ends, or ends with an API error” |
| `Notification` | “The agent sends a notification” |
| `SubagentStart`, `SubagentStop` | A subagent starts or finishes |
| `PreCompact`, `PostCompact` | “Conversation compaction runs” |

`matcher` is “a regular expression tested against the tool name (Claude tool names such as `Bash`, `Read`, and `Edit` are mapped to Grok’s automatically)”; `type` is `command` or `http` (“with a `url` to POST the event to”); “`timeout` is in seconds, default 5.”

**Trust:** “Project hooks require trust before they run: the first time you open a repo with hooks, grant it with `/hooks-trust` or by launching with `--trust`. The decision is stored in `~/.grok/trusted_folders.toml` and covers project MCP and LSP servers too.”

**Script contract:** the event arrives as JSON on stdin (`hookEventName`, `sessionId`, `cwd`, `workspaceRoot`, `toolName`, `toolInput`), and every hook process gets `GROK_HOOK_EVENT`, `GROK_HOOK_NAME`, `GROK_SESSION_ID`, `GROK_WORKSPACE_ROOT` in its environment; plugin hooks also get `GROK_PLUGIN_ROOT` and `GROK_PLUGIN_DATA`. A `PreToolUse` hook decides by writing `{"decision": "deny", "reason": "..."}` to stdout: “Exit code 0 allows, exit code 2 denies. Everything else — timeouts, crashes, malformed output — is fail-open: the failure is recorded in the session but the tool call proceeds. Only an explicit `deny` blocks. For passive events, stdout is ignored; exit 0 on success.” Loaded hooks are visible in the `/hooks` tab of the extensions modal.

## Skills

“Skills are reusable folders containing markdown instructions, script files, and resources for agents.” Discovery: `./.grok/skills/` (“walked up to the repo root”), `~/.grok/skills/`, “Any enabled plugin’s `skills/` directory,” and `[skills] paths` extras. “User-invocable skills also appear as slash commands, for example `/<skill-name>`”; on collisions “use the qualified form, such as `/local:commit`.”

`SKILL.md` frontmatter: `name` (“Directory name if omitted”), `description` (“First body paragraph if omitted”), `when-to-use` (alias `when_to_use`), `paths` (“Gitignore globs. Hidden until a matching file is touched”), `allowed-tools` — which “**Does not grant or restrict tools**” — `argument-hint`, `user-invocable` (default `true`; “Only the literal `true` counts; `yes` is false”; `false` hides it “from you and from the model”), `disable-model-invocation` (default `false`), and `metadata` (`author`, `short-description` shown in UI). “Grok accepts `model`, `effort`, `license`, and `compatibility` and does not apply them.” `[skills] paths` adds directories; `[skills] disabled` lists skills that are discovered “but [do] not activate.”

## Plugins and marketplaces

“Plugins extend Grok with additional skills, agents, hooks, MCP servers, and LSP servers.” Sources: `./.grok/plugins/`, `~/.grok/plugins/`, “Marketplace installs under `~/.grok/plugins/marketplaces/`”, `[plugins] paths`, and `--plugin-dir <PATH>`. `[plugins] enabled` exists because “project plugins may default off”; `[plugins] disabled` mirrors skills. “The TUI includes a Marketplace tab for browsing and installing plugins from configured sources” — `[[marketplace.sources]]` in `~/.grok/config.toml` plus `~/.grok/plugins/known_marketplaces.json`. CLI: `grok plugin <list|install|uninstall|update|enable|disable|details|validate>` and `grok plugin marketplace <list|add|remove|update>`. One extensions modal serves everything: “open it with any of `/plugins`, `/hooks`, `/skills`, or `/mcps`” (plus `/marketplace`), each pre-selecting a tab.

## MCP: three-plus scopes, `<server>__<tool>` naming

MCP tools surface “namespaced as `<server>__<tool>`.” Add with `grok mcp add` — stdio (“everything after `--` is the server command”) or HTTP (`--transport http`); `--header` is repeatable and `${VAR}` expansion works in headers. `grok mcp list|remove|doctor` manage servers (“`list` and `doctor` take `--json`”).

In TOML, `[mcp_servers.<name>]` supports stdio (`command`, `args`, `env`, `cwd`) and HTTP/remote (`url`, `headers`, `bearer_token_env_var`), with common `enabled`, `startup_timeout_sec` (default 30), `tool_timeout_sec` (default 6000), and `tool_timeouts` overrides. “Grok expands `${VAR}` (and `${VAR:-default}`) in `url`, `command`, `args`, `env`, and `headers`, so secrets can stay in the environment. Servers that require OAuth trigger a browser flow on first use; tokens are stored under `~/.grok/mcp_credentials.json`.” Headers may use `{{session_id}}`.

**Project scope:** “Pass `--scope project` to `grok mcp add` (it writes `.grok/config.toml` in the current directory) to define servers that ship with the repo. When loading, Grok walks from the current directory up to the git root reading each `.grok/config.toml`, and **a project server with the same name as a user one replaces it entirely**.” Vendor compatibility configs — `~/.claude.json`, `.cursor/mcp.json`, and project `.mcp.json` — are loaded “merged below `config.toml` in priority,” disableable per vendor with `[compat.claude] mcps = false` / `[compat.cursor] mcps = false`. “`grok inspect` shows every loaded server and its origin.” Troubleshooting: `grok mcp doctor` first; stdio stderr is captured to `~/.grok/logs/mcp/<server>.stderr.log`; cold-start `npx` servers “may need a higher `startup_timeout_sec`.” `GROK_TOOL_SEARCH` (default on) provides “On-demand MCP tool discovery for large toolsets.”

## Official sources

- [Hooks](https://docs.x.ai/build/features/hooks) · [Skills, Plugins & Marketplaces](https://docs.x.ai/build/features/skills-plugins-marketplaces) · [MCP Servers](https://docs.x.ai/build/features/mcp-servers)
- [Settings Reference](https://docs.x.ai/build/settings/reference) · [CLI Reference](https://docs.x.ai/build/cli/reference)
