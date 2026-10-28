# Google Antigravity corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| App, CLI, and IDE are one surface with identical settings paths. | ❌ FALSE | They share the agent/customization formats in places, but host settings and behavior differ. |
| CLI settings live in `~/.gemini/settings.json`. | ❌ FALSE | Current CLI settings use `~/.gemini/antigravity-cli/settings.json`; that legacy Gemini path is different. |
| App and CLI expose identical permission-mode names and defaults. | ❌ FALSE | App uses presets; CLI uses `agentMode` and `toolPermission` with its own defaults. |
| Every App preset enables the terminal sandbox. | ❌ FALSE | Default and Auto enable it; Request Review and Turbo do not. |
| CLI terminal sandbox is on by default. | ❌ FALSE | It is off by default; `auto` mode enables it automatically. |
| Turbo preserves workspace-only file restrictions. | ❌ FALSE | Turbo allows full-filesystem access and unrestricted commands, subject to explicit rules. |
| Allow rules override Deny rules. | ❌ FALSE | Deny takes precedence; CLI documents Deny > Ask > Allow. |
| `read_url` and `execute_url` are the same permission. | ❌ FALSE | Reading/navigation and interactive browser actuation are separate. |
| Current permission rules require a separate `unsandboxed(...)` grant. | ❌ FALSE | That belongs to the previous permission settings; current `command(...)` covers inside and outside sandbox execution. |
| Only the most specific Antigravity rule file is loaded. | ❌ FALSE | Rules are cumulative; specificity resolves conflicts. |
| `AGENTS.md` requires a `trigger` frontmatter field. | ❌ FALSE | It is plain always-on Markdown; modular files under `rules/` require `trigger`. |
| `model_decision` rules are always injected in full. | ❌ FALSE | Their path and description are indexed, and the agent reads full content on demand. |
| Rule includes have no size budget. | ❌ FALSE | Limits apply after includes: 24,000 bytes per file plus the aggregate token budget. |
| Subagents inherit the parent's full conversation context. | ❌ FALSE | They start with clean context. |
| An isolated subagent worktree prevents parent access. | ❌ FALSE | Parents retain access to child workspaces. |
| Misspelled subagent tool names are always harmless. | ❌ FALSE | Current docs warn they can cause the subagent process to hang. |
| App and CLI use the same global skill directory. | ❌ FALSE | App documents `~/.gemini/config/skills/`; CLI migration documents `~/.gemini/antigravity-cli/skills/`. |
| Every plugin manifest requires `name` on every surface. | ⚠️ PARTIAL | CLI management requires it; App/IDE can default to the folder name. |
| Antigravity supports only stdio MCP. | ❌ FALSE | It also supports remote Streamable HTTP; `url` is accepted as well as `serverUrl`. |
| MCP has no default timeouts. | ❌ FALSE | Defaults are 300 seconds for connection/tool listing and 180 seconds per call. |
| MCP tool allowlisting and execution approval are the same control. | ❌ FALSE | Exposure filters and permission rules are separate. |
| CLI hooks are loaded from `settings.json`. | ❌ FALSE | Use global/workspace `hooks.json`; a `hooks` settings key is retained but not loaded. |
| Hooks only run before and after tools. | ❌ FALSE | Session and model invocation events plus Stop are also documented. |
| Headless soft denial always produces a nonzero exit. | ❌ FALSE | The run can continue and exit 0 after an approval-required tool is soft-denied. |
| Headless exit 0 proves the task and tests passed. | ❌ FALSE | It means a response was produced; inspect status/error and run explicit validation. |
| Shared customization support proves full IDE parity. | ❓ UNVERIFIED | Verify each host's settings and lifecycle behavior rather than inferring it. |
