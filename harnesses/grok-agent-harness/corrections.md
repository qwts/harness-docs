# Grok Build corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| Configuration lives in `~/.config/grok/` as JSON. | ❌ FALSE | Settings are TOML “persisted under `~/.grok/config.toml` (on Windows, `%USERPROFILE%\.grok\config.toml`)”; `$GROK_HOME` overrides the home. |
| A project `.grok/config.toml` can set any setting. | ❌ FALSE | “Project `.grok/config.toml` only contributes **`[mcp_servers]`**, **`[plugins]`**, and **`[permission]`**. Other sections belong in user config.” |
| The sandbox is on by default. | ❌ FALSE | “(Landlock on Linux, Seatbelt on macOS). Off by default”; `GROK_SANDBOX` default is `off`. |
| The sandbox protects `~/.ssh` and similar credential paths. | ❌ FALSE | “Built-ins do not permanently protect paths such as `~/.ssh`; use a custom `deny` list.” |
| Sandbox network restrictions apply on macOS. | ❌ FALSE | “Child network — Enforced on Linux only; no-op on macOS for `read-only` / `strict`.” |
| Hook failures block the tool call. | ❌ FALSE | “Everything else — timeouts, crashes, malformed output — is fail-open: the failure is recorded in the session but the tool call proceeds. Only an explicit `deny` blocks.” |
| Many hook events can block tool calls. | ❌ FALSE | `PreToolUse` is “the only blocking event”; the reference lists 14 events, the rest passive. |
| Hook timeouts default to 30 or 60 seconds. | ❌ FALSE | “`timeout` is in seconds, default 5.” |
| Project hooks run without approval. | ❌ FALSE | “Project hooks require trust before they run… grant it with `/hooks-trust` or by launching with `--trust`”; stored in `~/.grok/trusted_folders.toml` covering “project MCP and LSP servers too.” |
| Deny rules stop applying under always-approve. | ❌ FALSE | Always-approve auto-approves tool calls but “(`deny` rules and PreToolUse hooks still apply).” |
| A remembered “always allow” grant covers dangerous commands like `rm` and `git push`. | ❌ FALSE | “A remembered ‘always allow’ grant still prompts for dangerous patterns such as `rm` and `git push`. An explicit config or CLI allow rule auto-approves them.” |
| `permission_mode` can be set in project config. | ❌ FALSE | “Default in user config only (`~/.grok/config.toml` or managed/requirements — not project `.grok/config.toml`).” |
| Claude Code’s `disableBypassPermissionsMode: "disable"` locks Grok’s always-approve. | ❌ FALSE | “Claude Code’s `managed-settings.json` `disableBypassPermissionsMode: "disable"` is **not** applied to grok’s always-approve”; grok honors that file’s “permission rules, MCP allowlists, and marketplace restrictions” only, and its own lock “is honored only from root-owned sources.” |
| Claude Code CLI flags are unsupported. | ❌ FALSE | “Claude Code flag names are accepted as aliases where they overlap: `--allowedTools`, `--disallowedTools`, `--append-system-prompt`, `--system-prompt`, and `--dangerously-skip-permissions`.” |
| Auto mode skips the plan review. | ❌ FALSE | “Auto and always-approve do not skip this review.” |
| Plan mode prevents all file writes. | ⚠️ PARTIAL | It gates edit tools only — “Plan mode gates edit tools, not the shell — bash can still write via redirection” — and “Subagents are not edit-gated by the parent’s plan mode; they do inherit permission mode.” |
| A project MCP server merges with a same-named user server. | ❌ FALSE | “A project server with the same name as a user one replaces it entirely.” |
| MCP tools are namespaced `mcp__server__tool`. | ❌ FALSE | Their tools are “namespaced as `<server>__<tool>`.” |
| Vendor MCP configs (`~/.claude.json`, `.cursor/mcp.json`, `.mcp.json`) take precedence over `config.toml`. | ❌ FALSE | They are “merged below `config.toml` in priority”; disable per vendor with `[compat.claude] mcps = false` / `[compat.cursor] mcps = false`. |
| A skill’s `allowed-tools` frontmatter restricts which tools it can run. | ❌ FALSE | “`allowed-tools` — Does not grant or restrict tools.” |
| `user-invocable: yes` hides a skill from slash commands. | ✅ TRUE | “Only the literal `true` counts; `yes` is false,” and `false` “hides it from you and from the model” — an explicit `yes` therefore hides the skill. Omitting the field is what yields the default `true`; the footgun is writing `yes` intending to expose it. |
| Grok reads only `AGENTS.md`. | ❌ FALSE | It reads “any of `AGENTS.md`, `Agents.md`, `AGENT.md`, `CLAUDE.md`, `Claude.md`, and `CLAUDE.local.md`, plus every `*.md` file in a `.grok/rules/` directory,” with `.claude/rules/` and `.cursor/rules/` for compatibility. |
| Rewind only changes the conversation. | ❌ FALSE | “Rewind modifies files on disk — reverted changes are lost unless committed to git.” |
| Headless runs leave no session on disk. | ❌ FALSE | “Sessions work the same in the TUI, headless mode, and over ACP,” under `~/.grok/sessions/` keyed by working directory. |
| `-s, --session-id` resumes an existing session. | ❌ FALSE | “`-s, --session-id` names a new session with a UUID you supply; it does not resume existing ones.” |
| `/loop` tasks run indefinitely. | ❌ FALSE | “Loops expire after 7 days, and at most 50 scheduled tasks can be active at once.” |
| Ending a session removes its worktree. | ❌ FALSE | “Worktrees persist until you remove them: ending or deleting a session leaves its worktree in place, and `gc` runs only when you invoke it.” |
| `XAI_API_KEY` always wins over config-stored keys. | ❌ FALSE | Resolution is per model: “`model.api_key` > `model.env_key` > active session token > `XAI_API_KEY`.” |
| `requirements.toml` settings can be overridden by env vars or user config. | ❌ FALSE | “Settings in `requirements.toml` cannot be overridden by lower layers, remote settings, or user config — use it for compliance-critical policies.” |
| `web_fetch` is enabled by default. | ❌ FALSE | `GROK_WEB_FETCH` defaults to `0` — “Off by default for security.” |
| The LSP tool is enabled by default. | ❌ FALSE | `GROK_LSP_TOOLS` default is `0` (“Enable the LSP code-intel tool (`1`/`0`)”). |
| Grok Bot is the cloud/web version of Grok Build. | ❌ FALSE | Grok Bot is a separate product — “your proactive AI teammate” on “a real computer in the cloud”; its documentation never mentions running Build sessions, and the Build docs list it only under “Next.” |
| Grok Build and Grok Bot share documentation scope. | ⚠️ PARTIAL | The Build overview links “Grok Bot — AI teammates on a cloud computer” as related, but the surfaces, config, permissions, and sessions documented here are CLI/TUI/ACP features with no documented Bot equivalent. |
| Always-approve is the recommended daily mode. | 💬 OPINION | The docs position `strict` + `dontAsk` for untrusted trees and keep Ask as the default; mode choice is workload-dependent. |
