# Permissions, sandbox, and plan mode

**Verified:** 2026-11-15

## Permission modes

“Permissions decide which tool calls may run. The sandbox is separate: it limits what an approved call can do on the filesystem and network.”

| Mode | Behavior | Enter via |
|---|---|---|
| Ask (default) | “Prompt for anything not already allowed” | — |
| Auto | “Classifier auto-approves safe tools; dangerous ones may still prompt (`deny` rules and hooks still apply)” | `/auto`, `Shift+Tab` when the feature is on |
| Always-approve | “Auto-approve tool calls (`deny` rules and PreToolUse hooks still apply)” | `/always-approve`, `Ctrl+O`, `Shift+Tab`, `grok --always-approve` |

“`Shift+Tab` cycles Normal → Plan → Auto (when available) → Always-approve.” `/auto` “only appears when the auto permission-mode feature is enabled,” and auto/always-approve switch rather than stack. The default comes from `[ui] permission_mode` in **user config only** (“~/.grok/config.toml` or managed/requirements — not project `.grok/config.toml`”). Legacy keys still work: “Legacy keys `approval_mode` and `yolo = true` still work; `permission_mode` wins when more than one is set.”

Enterprise headless modes: “`dontAsk` — Silently deny anything without an explicit allow rule (Headless, CI)” and “`acceptEdits` — Auto-approve file edits; prompt for shell commands.” Locking: `disable_bypass_permissions_mode = true` under `[ui]` “turns always-approve (bypass-permissions) off across the deployment,” blocking “the `--yolo` and `--permission-mode bypassPermissions` flags, the in-session toggles (Ctrl+O, `/always-approve`, and the Shift+Tab mode cycle), client-supplied yolo settings, and catch-all `allow` rules such as `*` or `**`.” Two tampering caveats: the lock “is honored only from root-owned sources (`/etc/grok/requirements.toml` or the system layer), not from the user-writable `~/.grok/requirements.toml`,” and “Claude Code’s `managed-settings.json` `disableBypassPermissionsMode: "disable"` is **not** applied to grok’s always-approve — grok honors that file’s permission rules, MCP allowlists, and marketplace restrictions, but a developer’s `--yolo` / `[ui] permission_mode` / runtime toggle still take effect.”

## Rules: deny > ask > allow

`[permission]` takes compact arrays — “Compact rules, for example `Bash(git *)`, `Read(src/**)`, `Edit(**/*.rs)`, `MCPTool(server__*)`” — or the verbose `{ action, tool, pattern? }` form. `--allow`/`--deny` “take the same patterns per invocation. Supported filters include `Bash`, `Edit`, `Read`, `Grep`, `MCPTool`, `WebFetch`, and `WebSearch`. `deny` always wins over `allow`.” Remembered UI grants are weaker than config: “A remembered ‘always allow’ grant still prompts for dangerous patterns such as `rm` and `git push`. An explicit config or CLI allow rule auto-approves them. Under always-approve they run unless you add a deny.”

## Sandbox: separate layer, off by default

“The sandbox limits what the agent process and its children can read, write, and reach on the network (Landlock on Linux, Seatbelt on macOS). Off by default.”

| Profile | Read | Write | Child network |
|---|---|---|---|
| `off` | Unrestricted | Unrestricted | Allowed |
| `workspace` | Everywhere | “CWD, `~/.grok/`, temp” | Allowed |
| `devbox` | Everywhere | “Top-level dirs except `/data`” | Allowed |
| `read-only` | Everywhere | “`~/.grok/` and temp only” | Blocked |
| `strict` | “CWD and system paths” | “CWD, `~/.grok/`, temp” | Blocked |

Documented limits: “Child network — Enforced on Linux only; no-op on macOS for `read-only` / `strict`”; “Credentials — Built-ins do not permanently protect paths such as `~/.ssh`; use a custom `deny` list”; “`~/.grok/` — Stays writable under sandboxed profiles so sessions can persist”; “In-process network — Model API and web tools are not blocked by child-network settings.”

Custom profiles in `~/.grok/sandbox.toml` or project `.grok/sandbox.toml` extend a built-in (`extends` default `workspace`) with `restrict_network`, `read_only`, `read_write`, and `deny` — “Kernel-enforced deny for read and write/rename. An entry is a glob if it contains `*`, `?`, or `[` (for example `**/.env`, `**/*.pem`).” On Linux, “read-deny requires `bubblewrap`.” Enable via `--sandbox`, `[sandbox] profile`, or `GROK_SANDBOX`; `GROK_SANDBOX_AUTO_ALLOW_BASH`/`[sandbox] auto_allow_bash` skip bash prompts while a profile is active. “Built-in names (`off`, `workspace`, `read-only`, `strict`, `devbox`) cannot be redefined as custom profiles.” For untrusted trees: “pair a strict profile with narrow permission allows (or headless `dontAsk`).”

## Plan mode

“In plan mode the agent explores the codebase and drafts a plan for your approval before it edits anything.” Enter with `/plan [description]` or `Shift+Tab`; “The agent can enter plan mode on its own when a task looks ambiguous. That is not a permission prompt.” The review surface (a=approve, s=request changes, c=comment, q=quit) “is not skipped under auto or always-approve,” and “Auto and always-approve do not skip this review.”

Caveats, quoted: “Only the session plan file may be edited until you approve. Other edit tools are rejected, including under auto or always-approve. Reads, bash, and MCP still follow permission mode. Plan mode gates edit tools, not the shell — bash can still write via redirection.” And: “Subagents are not edit-gated by the parent’s plan mode; they do inherit permission mode (including auto and always-approve).” Status shows `plan` / `plan approval`; the mode stays on until approve or quit.

## Official sources

- [Permissions](https://docs.x.ai/build/features/permissions) · [Sandbox](https://docs.x.ai/build/features/sandbox) · [Plan Mode](https://docs.x.ai/build/features/plan-mode)
- [Settings Reference](https://docs.x.ai/build/settings/reference) · [Enterprise Deployments](https://docs.x.ai/build/enterprise) · [Modes and Commands](https://docs.x.ai/build/modes-and-commands)
