# Tools, approvals, permissions, and sandbox

**Verified:** 2026-11-16 (Qwen Code 0.27.4 npm latest; docs site main)

## Built-in tools

The tool reference documents file-system tools (`read_file`, `write_file`, `edit`), `read_many_files` (multi-file), `run_shell_command` (shell), the MCP tool, `memory`, `monitor` (deferred by default), `task` (the Agent tool), `todo_write`, `web_fetch`, `web_search`, `exit_plan_mode`, and sandbox customization. Two experimental tool surfaces change how tools are exposed: “Set `tools.codeModeOnly` to `true` to enable experimental Code Mode with lazy tool discovery” (tools invoked through Code Mode's `exec`), and the deferred-tool bridge — “tools deferred by default (such as `task_stop` or `monitor`) keep their usual on-demand loading via the `tool_search` + `tool_call` bridge.” “To keep a tool's schema out of the initial model request, use `tools.eager`; to block a tool outright, use a whole-tool `permissions.deny` rule.”

## The five approval modes

“Qwen Code offers five distinct permission modes” (`tools.approvalMode`, cycled with Shift+Tab, or `--approval-mode` at launch):

| Mode | File editing | Shell commands |
|---|---|---|
| **Plan** | “❌ Read-only analysis only” | “❌ Not executed” |
| **Ask Permissions** | manual approval | manual approval |
| **Auto-Edit** | “✅ Auto-approved” (`edit`, `write_file`, `notebook_edit`) | manual approval |
| **Auto** | “✅ Classifier-evaluated” | “✅ Classifier-evaluated” |
| **YOLO** | auto-approved | auto-approved |

Notes: “The mode previously named **Default** has been renamed to **Ask Permissions** … The underlying configuration value (`tools.approvalMode: "default"`) … unchanged.” Plan Mode: `/plan`, `/plan exit` (“your previous approval mode is automatically restored”), Shift+Tab cycling (“⏸ plan mode”). YOLO: “`--approval-mode yolo` … (equivalent to `--yolo`) … Cannot be used together with `--yolo`”; `/approval-mode yolo --project` or `--user` persists it. `--allowed-tools` is “A comma-separated list of tool names that will bypass the confirmation dialog.”

## Auto Mode (classifier)

“When you're in Auto Mode and the agent tries to run a tool, Qwen Code walks three layers in order”:

1. “**acceptEdits fast-path** — Edit / Write whose target path is inside the workspace is auto-approved without invoking the classifier. **Exception:** writes to Qwen Code's own self-modification surfaces (`.qwen/settings*.json`, `QWEN.md`, `AGENTS.md`, `QWEN.local.md`, configured context filenames, `.qwen/rules/`, `.qwen/commands/`, `.qwen/agents/`, `.qwen/skills/`, `.qwen/hooks/`, `.mcp.json`) and persistence surfaces (`.git/`, `.husky/`, `package.json`, `.npmrc`, `Makefile`, `.github/workflows/`, etc.) route through the classifier … Symlinks targeting protected paths are resolved and rejected too.”
2. “**Safe-tool allowlist** — Read-only and metadata-only built-in tools (Read, Grep, Glob, LS, LSP, TodoWrite, AskUserQuestion, etc.) are auto-approved.”
3. “**LLM classifier** — Everything else (shell commands, web fetches, sub-agent spawns, edits outside the workspace, MCP tools) is sent to a two-stage classifier:” “**Stage 1 (fast)** — outputs `{ shouldBlock }` only. Around ~300ms”; “**Stage 2 (thinking)** — only runs when Stage 1 said block … Can downgrade Stage 1's block to allow.”

“The classifier uses your configured fast model (`/model --fast`).” Stage 1 has a 10-second default budget; Stage 2 uses `permissions.autoMode.classifier.timeouts.stage2Ms` (“30 seconds by default”), with a session-model fallback review before manual approval. Guardrails: `permissions.deny` rules win before the classifier; “over-broad allow rules are stripped while in Auto Mode”; “**Fail-closed**: when the classifier API is unreachable, the action is blocked rather than allowed”; “**Loop guard**: after three consecutive blocks … the call that reaches the third block also falls back to manual approval.” Hints: `permissions.autoMode.hints.allow`/`deny`, `environment`, `classifyAllShell: true` (“route ALL shell commands … through the classifier”), and `mcp.forwardArguments: false` (“send MCP tool calls to the classifier by name only”).

Subagents: “which run in Auto-Edit by default when the session is in Ask Permissions mode in a trusted folder: a foreground subagent asks inline, and a background subagent, which cannot ask, has the call denied.”

## Permission rules

“The permissions system provides fine-grained control … **Decision priority (highest first): `deny` > `ask` > `allow`**” (then the default/interactive mode). “The first matching rule wins. Rules use the format `"ToolName"` or `"ToolName(specifier)"`.” The three arrays are “Merged across all scopes (user + project + system)” and “this never affects whether a tool is registered” (for `allow`/`ask`).

- Aliases: `Bash`/`Shell` → `run_shell_command`; `Read`/`ReadFile` → `read_file`; `Edit`/`EditFile` → `edit`; `Write`/`WriteFile` → `write_file`; `NotebookEdit` → `notebook_edit`; `Grep`/`SearchFiles` → `grep_search`; `Glob`/`FindFiles` → `glob`; `ListFiles` → `list_directory`; `WebFetch` → `web_fetch`; `Agent` → `task`; `Skill` → `skill`.
- Meta-categories: `Read` covers “`read_file`, `grep_search`, `glob`, `list_directory`”; `Edit` covers “`edit`, `write_file`, `notebook_edit`.” “To restrict only file reading, use `ReadFile(/path/**)` or `read_file(/path/**)`.”
- Path prefixes: `//` absolute, `~/` home-relative, `/` project-root-relative, `./` cwd-relative, none = `./`.
- **Shell bypass prevention:** “Permission rules for `Read`, `Edit`, and `WebFetch` are also enforced when the agent runs equivalent shell commands … Supported shell commands include `cat`, `grep`, `curl`, `wget`, `cp`, `mv`, `rm`, `chmod`, and many more. Unknown/safe commands (e.g. `git`) are unaffected.”
- Registry effects: “A whole-tool deny rule (no specifier) also removes the tool from the registry — for built-in tools and tools found via `tools.discoveryCommand`. MCP tools are exempt … Deny rules still block MCP tool calls at runtime.”
- AUTO-mode stash: “under the AUTO approval mode, dangerous allow rules are stashed rather than active … when AUTO mode is exited the stashed rule is restored and auto-approves again until the session restarts.”
- Legacy `tools.allowed`/`tools.exclude`/`tools.core` are “still honoured at startup” but not auto-migrated; “An empty `tools.core` list is treated as unset and disables nothing.”
- “Use `/permissions` in the interactive CLI to view, add, and remove rules without editing `settings.json` directly.”

## Folder trust

“**The Trusted Folders feature is disabled by default.** While it is disabled, every folder counts as trusted, so a project's `.qwen/settings.json` is loaded without any prompt.” Enable with `"security": {"folderTrust": {"enabled": true}}`. The dialog offers “**Trust folder** … **Trust parent folder** … **Don't trust**”; decisions are saved in `~/.qwen/trustedFolders.json` (“you will only be asked once per folder”). “The feature fails closed: until you make a choice, a folder counts as **untrusted**.” Trust order: “**IDE Trust Signal:** … The IDE's response takes highest priority”, then the local trust file. An untrusted folder runs in a restricted “safe mode”: workspace settings ignored, `.env` files ignored, “You **cannot install, update, or uninstall** extensions”, “You will always be prompted before any tool is run”, and “Automatic Memory Loading is Disabled.” Even with the feature off, “Project MCP servers and project hooks still ask,” and “Disabled folder trust does not authorize project skill/subagent frontmatter hooks, project skill automatic tool grants, or workspace `.lsp.json` servers.”

## Sandbox

Two distinct mechanisms:

**Linux tool execution sandbox** (`tools.executionSandbox`) — “an operator-only Linux policy with required `filesystem: "read-only" | "workspace-write"` and `network: "open" | "closed"`, plus optional `backend: "auto" | "bwrap" | "landlock"`.” “Install `bwrap` (Bubblewrap) at `/usr/bin/bwrap` … for the strongest supported boundary”; “`auto` prefers bwrap and can fall back to partial Landlock filesystem enforcement only when command networking is open. … Landlock cannot enforce `network: closed`.” “System policy replaces the complete User/SystemDefaults object; Workspace values are ignored … even in a trusted project.” Scope: “The CLI, model transport, authentication and session storage remain on the host. Shell, terminal `!`, prompt shell interpolation, Monitor, Read/Write/Edit and supported nested in-process Agent/Code Mode calls use the same runtime policy.” Boundaries: “Approval and YOLO do not expand the filesystem or network policy”; “Reads remain broad: this mode does not hide host secrets”; malformed operator policy “blocks startup (configuration exit code 52)”. “ACP, `qwen serve` and web terminals explicitly reject it,” and “file checkpoints … automatic skill/memory maintenance … are disabled or rejected” in this mode. Verify with `qwen sandbox`, `qwen sandbox --verify`, `qwen sandbox -- sh -c '...'`.

**Whole-CLI sandboxing** — macOS Seatbelt (“Lightweight, built-in sandboxing using `sandbox-exec`”; profiles `permissive-open` (default), `permissive-closed`, `permissive-proxied`, `restrictive-open`, `restrictive-closed`, `restrictive-proxied` via `SEATBELT_PROFILE`; custom profiles at `.qwen/sandbox-macos-<profile_name>.sb`), and Docker/Podman containers (“mounts your workspace and your `~/.qwen` directory into the container so auth and settings persist”). Enable precedence: environment variable `QWEN_SANDBOX=true|false|docker|podman|sandbox-exec`, then the `-s`/`--sandbox` flag, then `tools.sandbox` — “If `QWEN_SANDBOX` is set, it **overrides** the CLI flag and `settings.json`.” Image precedence: “`--sandbox-image` > `QWEN_SANDBOX_IMAGE` > `tools.sandboxImage` > built-in default image” (`ghcr.io/qwenlm/qwen-code:<version>`). Extend via `.qwen/sandbox.Dockerfile` / `.qwen/sandbox.bashrc`; `SANDBOX_FLAGS` injects container flags; network allowlisting via `QWEN_SANDBOX_PROXY_COMMAND` (proxy on `:::8877`); Linux UID/GID mapping via `SANDBOX_SET_UID_GID`. Security framing: “Sandboxing reduces but doesn't eliminate all risks.”

`--safe-mode` (“Disable all customizations — context files, hooks, extensions, skills, MCP servers, custom subagents (only built-in subagents load), permission rules, settings-sourced approval mode overrides, memory features, cross-session messaging, LSP, and sandbox settings …; the CLI flags `--yolo` and `--approval-mode` still take effect”) and `--bare` disable local customizations; “Workspace-only recovery remains available without operator confinement.”

## Official sources

- [Approval Mode](https://qwenlm.github.io/qwen-code-docs/en/users/features/approval-mode/) · [Auto Mode](https://qwenlm.github.io/qwen-code-docs/en/users/features/auto-mode/) · [Sandbox](https://qwenlm.github.io/qwen-code-docs/en/users/features/sandbox/) · [Trusted Folders](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/trusted-folders/) · [Settings → permissions](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/) · [Tools reference](https://qwenlm.github.io/qwen-code-docs/en/developers/tools/introduction/) · [LSP](https://qwenlm.github.io/qwen-code-docs/en/users/features/lsp/)
