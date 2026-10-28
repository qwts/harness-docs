# Cline corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| `.clinerules/` is the only current project configuration root. | ❌ FALSE | Current shared project configuration uses `.cline/`; `.clinerules/` remains supported for rules and extension-specific hooks. |
| `~/.cline/data/` is the global configuration root. | ❌ FALSE | `~/.cline/` is the configuration root; `~/.cline/data/` holds application state. |
| `CLINE_DIR` changes only application data. | ❌ FALSE | It replaces `~/.cline/`; `CLINE_DATA_DIR` separately replaces `~/.cline/data/`. |
| Only one of `.clinerules/` and `.cline/rules/` is searched. | ❌ FALSE | Both are searched when present. |
| Cline reads only files named `AGENTS.md`. | ❌ FALSE | It also documents rule directories and compatibility files including `.cursorrules` and `.windsurfrules`. |
| Global rules always override workspace rules. | ❌ FALSE | Workspace rules override global rules on conflict. |
| Every rule requires YAML frontmatter. | ❌ FALSE | A rule without frontmatter is always active. |
| `paths: []` means a rule has no path restriction and always runs. | ❌ FALSE | It means the rule never activates. |
| Invalid conditional-rule YAML disables the rule safely. | ❌ FALSE | Parsing fails open: the rule activates and raw content is visible. |
| `.clinerules/` applies independently to every root in a multi-root IDE workspace. | ❌ FALSE | It applies only to the primary workspace. |
| Project skills override same-named global skills. | ❌ FALSE | The global skill overrides the project skill. |
| A skill’s `name` can differ from its directory name. | ❌ FALSE | It must exactly match the directory name. |
| Skill instructions and all resources are always loaded at startup. | ❌ FALSE | Skills use progressive loading: metadata, then instructions when triggered, then resources as needed. |
| Plan mode guarantees that no write can occur. | ❌ FALSE | It restricts normal editing and some shell commands but is not a non-write guarantee. |
| One-shot CLI prompt runs use interactive approval defaults. | ❌ FALSE | They default to Act mode with auto-approval enabled. |
| Auto-approval defaults to true in ACP mode too. | ❌ FALSE | ACP changes the `--auto-approve` default to false. |
| Cline CLI has a built-in shell allow/deny list. | ❌ FALSE | Current docs recommend a `PreToolUse` hook to block `run_commands`. |
| `.clineignore` remains a complete built-in secrecy boundary. | ❌ FALSE | Built-in behavior is deprecated; the hook replacement has search, shell-parsing, and symlink limitations. |
| CLI and VS Code workspace hooks use the same directory and filenames. | ❌ FALSE | CLI uses `.cline/hooks/`; VS Code uses `.clinerules/hooks/` with exact extensionless executable event files. |
| CLI file hooks still enforce policy in `--yolo` mode. | ❌ FALSE | CLI file hooks are disabled in that mode. |
| Plugins are available in VS Code and JetBrains. | ❌ FALSE | Current plugin support applies to SDK, CLI, and Kanban. |
| A Cline package plugin requires a root `plugin.json`. | ❌ FALSE | It uses a `cline` field in `package.json`. |
| Plugin worker threads form a security sandbox. | ❌ FALSE | They provide crash isolation, not a security boundary. |
| Reloading a plugin replaces code in a running session. | ❌ FALSE | Plugins bind at session start; running sessions retain their original code. |
| Plugin hook failures always stop the run. | ❌ FALSE | Default `hookErrorMode` is `"ignore"`; `"throw"` fails the run. Repeated failures can degrade or disable a plugin. |
| Omitting an MCP remote `type` selects Streamable HTTP. | ❌ FALSE | Omission defaults to legacy `sse`; specify `streamableHttp` explicitly. |
| Cline definitely uses `.cline/mcp.json` for project MCP. | ❓ UNVERIFIED | This research did not establish a project-local MCP file; use the documented CLI settings path or surface-specific UI. |
| Subagents are available in all Cline IDE extensions. | ❌ FALSE | They are documented for CLI and Desktop, not VS Code or JetBrains. |
| Subagents inherit MCP tools automatically. | ❌ FALSE | They do not receive MCP tools. |
| Every subagent tool call independently prompts for approval. | ❌ FALSE | Approving `spawn_agent` approves subsequent child tool activity. |
| Subagents and agent teams are the same feature. | ❌ FALSE | Teams add a coordinator, task board, mailbox, mission log, and separate persistent state. |
| Agent teams work with `--yolo`. | ❌ FALSE | Teams are unavailable with `--yolo`. |
| CLI JSON output is one JSON document containing the final answer. | ❌ FALSE | Agent runs emit newline-delimited JSON events; select `run_result` for the final answer. |
| A default CLI timeout eventually terminates every stuck run. | ❌ FALSE | Default timeout `0` means no timeout. |
| `@cline/agents` includes the full built-in ClineCore environment. | ❌ FALSE | It is a lighter stateless loop without built-in tools by default; `@cline/core` provides the full harness. |
| Old XML-style tool names are the current ClineCore tool contract. | ❌ FALSE | Current built-ins use names such as `read_files`, `run_commands`, and `apply_patch`. |
| Shared configuration proves full parity across IDE, Desktop, CLI, cloud, and SDK. | ❌ FALSE | Plugins, teams, subagents, hooks, paths, and approval defaults differ by surface. |
