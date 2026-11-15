# Tools, MCP, skills, and hooks

**Verified:** 2026-11-15

## MCP servers

“Junie CLI uses the same MCP JSON configuration as Junie in JetBrains IDEs” — a standard `mcpServers` object in:

- Project scope: `.junie/mcp/mcp.json` (“can be checked into version control”; “avoid sharing secrets … if the … file is committed”)
- User scope: `~/.junie/mcp/mcp.json`

Connection types: **Remote** (HTTP/HTTPS hosted servers, with an OAuth “Authorization required” status and a → Authorize browser flow) and **Local** (Docker, npx, or binary). Server statuses: Starting / Active / Inactive / Disabled / Failed / Authorization required — “The Inactive status indicates that the server is correctly configured and enabled but is not currently running”; Failed covers connect/crash errors. The `/mcp` screen lists, adds (Ctrl+A opens the pre-configured registry and the **MCP Installation Assistant** — an AI helper that configures registry servers or “searches the official MCP registry,” prompts for secrets, writes `mcp.json`, “and verifies the server startup”), enables/disables, reconnects, and authorizes. In ACP clients, `/mcp` “is a read-only listing” of local servers (with status and source) plus any servers the client passed at session init.

**Timeout:** “By default, Junie CLI waits up to 60 seconds for an MCP server to respond” — startup handshake, tool listing, and every call. A per-server optional `timeout` field (milliseconds, 1,000–3,600,000) overrides it; “Values outside this range are ignored, a warning is logged, and the 60-second default is used. Note that the value is in milliseconds… `"timeout": 60` is rejected instead of being treated as one minute.”

Discovery flags: `--mcp-location` (repeatable) and `--mcp-default-locations` (default true), mirrored by `mcp-locations`/`mcp-default-locations` in config.json. The IDE plugin page adds: “To effectively choose among the available tools, Junie supports a maximum of 100 tools from all configured MCP servers,” and notes (for IDE configs) that “At the moment, Junie does not support security tokens in MCP configs. As a workaround, you can load environment variables from Docker’s .env file.”

## Agent skills

“Skills follow the open Agent Skills format and are portable across agents.” Locations: `.junie/skills/<name>/` (project), `~/.junie/skills/` (user), plus “`.agents/skills/` directories” (`<projectRoot>/.agents/skills/` in a trusted project and `~/.agents/skills/`, “so skills shared through the cross-agent .agents convention are picked up as well”), extension-provided skills, and built-ins. Project-level beats user-level on name conflicts. Defaults (including `.agents/skills/`) are disableable with `--skill-default-locations false`; `--skill-location` adds folders.

Each skill folder needs a `SKILL.md` (“A folder without it is not recognized as a skill”) with YAML frontmatter `name` (required) and `description` (optional — otherwise “Junie CLI extracts the first paragraph of the body content… If the body is also empty or contains only headings, the skill will fail to load”), plus optional `scripts/`, `templates/`, `checklists/` subdirectories. Skills are invoked automatically by task relevance, explicitly as `/<skill-name>` slash commands, or via `$<skill-name>` prompt references (“Unlike guidelines, which are applied with every prompt, agent skills are only invoked when they match the needs of the current task” — progressive disclosure: “each skill’s name and description are available to Junie, so it knows what skills exist, but doesn’t read the full content of a skill until it determines its relevance”). `/skills` manages them: enable/disable persists across sessions; disabled skills are “excluded from automatic selection, from the `/<skill-name>` commands, and from the `$` suggestions, and their instructions are not added to Junie’s context.”

Cross-agent import: “if your project already has skill folders from other agents (.cursor/skills/, .claude/skills/, or .codex/skills/), Junie CLI will detect them and suggest importing into Junie’s .junie/skills/ directory.” Community skills from `github.com/JetBrains/skills` are copied in manually, with third-party-skill caution guidance (“Only use skills from sources you trust… Prefer pinning to a specific commit or tag”).

## Custom subagents (EAP configuration)

“Subagents in Junie extend the built-in logic of the main agent with task-specific instructions that define a tailored system prompt, tool restrictions, and usage of models and agent skills.” “Junie CLI invokes subagents automatically, informing the user which subagent has started”; the main agent “selects the relevant subagent by matching its name and description to the task at hand, runs it as a subtask, and brings the result back into the main session” — “Unlike custom slash commands, custom subagents cannot be invoked manually via slash commands.”

Files: Markdown with YAML frontmatter in `<projectRoot>/.junie/agents/`, `<projectRoot>/.agents/`, `~/.junie/agents/`, and `~/.agents/`. Frontmatter: `name` (defaults to filename; must match `[a-z][a-z0-9_-]*`), `description` (required — the delegation signal), `tools` (allowlist of tool groups), `disallowedTools` (denylist, applied after), `mcpServers` (MCP-server allowlist; “An empty or omitted list keeps every configured MCP server available”), `model`, `permissionMode` (`default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan`), `reasoningLevel` (`effort` is an accepted alias that “takes precedence if both are present”), `maxTurns`, `skills`, `allowPromptArgument`. Built-in tool groups: Read, Bash, Glob, Grep, Write, Edit, WebSearch, WebFetch, AskUserQuestion.

Standalone: `--agent <name>` / `JUNIE_AGENT` runs “every regular prompt in the session through the named custom agent without consulting the task router” — locks the session to Classic mode (“cannot be combined with `--agent-mode chat`”), while built-in workflows (`--review`, `/plan`, direct shell commands) “still route to their dedicated built-in handlers”; frontmatter model/effort/maxTurns override `--model`/`--effort`/`--max-steps`; an unknown name fails immediately listing available agents. Agent files from `.cursor/agents/`, `.claude/agents/`, `.codex/agents/` are detected with an import suggestion.

Model policy (EAP, `/settings` → Subagents, saved in settings.json): **SameModelOnly** — “may use subagents only for independent work that can run in parallel… Subagents always use the same model as the main agent”; **Auto** (default) — “may use SameModelOnly-style parallelism and may choose cheaper capable model tiers for suitable delegated work when it does not slow the session.”

## Hooks (EAP)

“Hooks let you run shell commands automatically at well-defined points in a Junie CLI session.” Configure a `hooks` object in `~/.junie/config.json` or an explicit `--config-location` file: “Project-local hooks from `<project-root>/.junie/config.json` are ignored by default for safety… If you intentionally want to run hooks from a project file, pass that file explicitly with `--config-location`.” Only `type: "command"` is supported; commands run via `sh -c` (macOS/Linux) or `cmd /c` (Windows) with one line of JSON on stdin.

| Event | Matcher target | Sync defaults |
|---|---|---|
| `SessionStart` | source: `startup`, `resume`, `clear`, `compact` | 10 s; failures never abort startup |
| `UserPromptSubmit` | none (every prompt; TUI only) | 10 s; can block the prompt (`decision: "block"`) |
| `PreToolUse` | tool name (Bash, Write, Read, Edit, Glob, Grep, `apply_patch`) | 10 s; allow/ask/block; `updatedInput` replaces input; exit 2 blocks |
| `PostToolUse` | tool name (add `apply_patch` to matchers — `Edit\|Write` alone misses it) | 600 s; feedback-only; fires only on successful tool calls |
| `Stop` | none (every transition; not chat-type tasks) | 600 s; exit 2 or `{"decision":"block","reason":…}` retries with feedback; `{"continue":false,"stopReason":…}` hard-halts; block cap 8 |
| `StopFailure` | error: `rate_limit`, `authentication_failed`, `billing_error`, `invalid_request`, `server_error`, `max_output_tokens`, `unknown`, `model_refused`, `country_forbidden` | 60 s (10 s dispatch budget SessionEnd, 60 s StopFailure); observability-only — “Output and exit code from the hook process are ignored” |
| `PermissionRequest` | tool category (Bash, Edit, Read, or MCP tool name) | 10 s; exit 0 **auto-approves**, exit 2/block auto-denies, other failures mean Junie “falls back to showing the normal permission dialog” |
| `SessionEnd` | reason: `prompt_input_exit`, `other`, `logout` | 2 s per command, 10 s total budget; “cannot block session termination” |

Junie extensions on the hook command: `timeout` (per-event defaults above), `strictJsonOutput` (non-empty stdout must be a JSON object), `blockOnError` (“promoted to a block” on failures — honored “for synchronous PreToolUse and Stop hooks only”), and `async: true` (runs in background; “decision, permissionDecision, and continue in the hook’s output have no effect”; `additionalContext` is “queued in memory and prepended to the agent prompt on the next user submit”). Default (non-`blockOnError`) failure semantics fail open: “Other non-zero exit codes are logged as a warning and the tool proceeds normally by default.”

Claude Code compatibility: “The field names match Claude Code’s StopFailure wire protocol, so the same hook script can be shared between agents”; PreToolUse/PostToolUse speak `decision`/`tool_input`-style JSON with exit-code-2 blocking; PostToolUse feedback reaches the agent “wrapped in `<post_tool_use_hook_feedback>` tags and marked as coming from a hook you configured, so the agent treats it as your instruction. Tool output cannot produce this message.”

Hosts: interactive TUI (all 8 events; UserPromptSubmit TUI-only), batch/`-p` (all but UserPromptSubmit, PermissionRequest), ACP (SessionStart, PreToolUse, SessionEnd only). “The Server host has been removed.” Stop-hook runaway guard: “the agent stops dispatching the Stop hook after 8 consecutive blocks within the same task and a system message is published. Override the limit with the JUNIE_STOP_HOOK_BLOCK_CAP environment variable; set it to 0 to disable the cap.”

Merging: entries from all trusted files concatenate (“Higher-priority files do not override lower-priority files”); order is config files first, then extension hooks in enablement order; skipped project hooks are “reported as a hook configuration warning.”

## Extensions

“Extensions are reusable bundles… A single extension can package any combination of:” agent skills, MCP servers, subagents, custom slash commands, guidelines, and hooks. `/extensions` (aliases `/plugin`, `/plugins`) browses marketplaces and installs at project scope (`.junie/extensions.json`, committable) or user scope (`~/.junie/extensions/extensions.json`); content caches once under `~/.junie/extensions/<marketplace>/<extension>/` and “Newly installed extensions become available within the running session.” Marketplaces may be a git repository, a local directory, or a direct URL to a manifest, in either “the native Junie format at `.junie-extension/marketplace.json`” or “the Claude plugin format at `.claude-plugin/marketplace.json`” — “you can connect any Claude-compatible plugin marketplace to Junie CLI.” The built-in JetBrains marketplace (`github.com/JetBrains/junie-extensions`) cannot be removed. Git/URL marketplace content updates only via → Update; local-directory marketplaces auto-refresh on start when the source changes. `hooks/hooks.json` in an extension follows the “Claude plugin layout,” with `${CLAUDE_PLUGIN_ROOT}` / `${JUNIE_EXTENSION_ROOT}` expanding to the cache path; “Extension hooks become active only after the extension is installed through /extensions… They are not loaded from an untrusted project-local config.json.”

## Official sources

- [Add and configure MCP servers](https://junie.jetbrains.com/docs/junie-cli-mcp-configuration.html) · [Agent skills](https://junie.jetbrains.com/docs/agent-skills.html) · [Custom subagents](https://junie.jetbrains.com/docs/junie-cli-subagents.html) · [Hooks](https://junie.jetbrains.com/docs/junie-cli-hooks.html) · [Add and configure extensions](https://junie.jetbrains.com/docs/junie-cli-extensions.html) · [Junie IDE plugin (MCP/allowlist sections)](https://junie.jetbrains.com/docs/junie-ide-plugin.html)
