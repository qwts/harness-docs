# Permissions, approvals, and trust

**Verified:** 2026-11-15

## Three permission modes

The interaction model has three approval modes. Always Ask (formerly Manual) is the default: “read-only tools (such as Read, Grep, Glob) are automatically allowed by default, while write and execution tools (such as Write, Edit, Bash) require user approval by default.” Ask When Needed (formerly YOLO, `/yolo` or `--yolo`) “auto-approves regular tool calls, making it suitable for batch tasks you know are safe” — “It still asks before sensitive actions,” and the docs list writing `.env` files, SSH private keys, and dangerous commands as still prompting. “Never Ask mode (formerly Auto), enabled with /auto, is the fully unattended mode: every tool approval is handled automatically, including sensitive files, and the agent never asks you questions — it decides everything on its own.” Only the Plan review still waits in Never Ask. `default_permission_mode` (manual / yolo / auto) picks the startup mode; “In -p mode, no human approval is requested — regular tool calls are handled under the auto permission policy, while static deny rules remain in effect.”

MCP calls inherit the modes — “In Ask When Needed mode, MCP tool calls are automatically approved. Only use this mode when you fully trust the MCP servers you have connected.” Session-scoped approvals offer “Approve for this session” so similar calls stop prompting until the session ends (the VS Code card words it “Yes, for this session”).

## Permission rules

`[[permission.rules]]` entries pair a tool `pattern` (`ToolName` or `ToolName(arg-pattern)`, with `mcp__<server>__<tool>` support and wildcard `*`) with a `decision` — “allow (permit immediately), deny (reject immediately), ask (prompt each time)” — and a `scope` (turn-override / session-runtime / project / user). Conflict resolution is fixed: “matching deny rules take precedence over matching ask rules, which take precedence over matching allow rules.” Project rules come from `.kimi-code/local.toml` — “Project rules are read from .kimi-code/local.toml only in the local environment when the workspace is trusted” — and “Remote project rules are not supported.”

The escape hatch is explicit and scoped: `dangerous_command_guard = false` under `[permission]` turns “off every built-in guard confirmation policy entirely: dangerous commands, access to sensitive files such as .env or id_rsa, and access to git control paths no longer ask for confirmation in Always Ask and Ask When Needed mode (these policies are never active in Never Ask mode); the default is true.” `KIMI_CODE_DANGEROUS_COMMAND_GUARD=false` overrides the file setting. Two things survive the switch: “With all built-in guards off, only ask rules you configured yourself can still prompt in Ask When Needed mode,” and “The hard block that stops Read/Write/Edit from touching secret files stays in effect.” The docs close with “Use this switch only for environments that already gate commands outside the agent.”

## Workspace trust

The first launch in a new directory shows a trust prompt before project-level configuration loads. “When Kimi Code finds project-level MCP servers in an untrusted folder, it shows each server's transport and launch target in the workspace trust prompt.” `[workspace].require_trust` (default true): “false treats every workspace as trusted and skips the startup trust prompt” — “Disabling the trust requirement loads project-level MCP servers and other trust-gated project configuration without confirmation.” The per-process `KIMI_CODE_TRUST_WORKSPACE=1` “does not write a persistent trust record.” Trust also gates headless runs: “Headless runs (for example kimi -p in CI) cannot show the trust prompt, so project-level MCP servers stay disabled there unless the workspace is already trusted.”

## Hooks

Hooks are shell commands wired to lifecycle events — “All hook rules are written in the [[hooks]] array in ~/.kimi-code/config.toml” (user config and plugins only; “there is no project-level hook file, so content checked out on a remote target can never inject hook definitions into your session”). Each rule has `event`, optional `matcher` (glob), `command`, optional `timeout` (seconds), `working_dir`, and `run_in_background`; “[[hooks]] only allows these four fields; extra fields will cause the config file to fail to load.” Outcomes by exit code: “Exit code: 0 means allow, 2 means block, other non-zero values default to allow” — and stdout JSON can return structured decisions. Matching runs concurrently: “When multiple rules match the same event, all matching hooks run in parallel; multiple rules with identical command values run only once.” Roughly twenty events exist across session, prompt, model, tool, subagent, file-watch, permission, and MCP lifecycle; “Only blockable events (PreToolUse, Stop, UserPromptSubmit) have return values that affect the main flow.” Hooks fail open — a crashing hook does not block the run. “hooks always execute on the machine running Kimi Code,” so in remote environments hooks never run on the remote host.

## Plan mode

Plan mode is toggled with Shift-Tab or `/plan [on|off|clear]`. “Only you can turn Plan mode on; the agent never enters it on its own, even when you ask it to in chat.” Inside it, “Write and Edit are restricted to writing the current plan file only, and TaskStop is blocked entirely.” Exit is the hard gate: “Exiting Plan mode always requires your confirmation, in every permission mode, including Never Ask.” The ExitPlanMode options the model can offer are constrained — labels ≤80 characters, and reserved words like Approve/Reject are rejected.

## Goal mode

`/goal` runs an objective-driven loop with statuses complete / paused / blocked. Objectives are ≤4000 characters and phrased as “what must become true.” Budget ticks are conservative: “the time budget only ticks while the goal is active and the session is open.” The `/goal next` queue lives in `upcoming-goals.json` and is invisible to the agent until promoted. Non-interactive `/goal` commands exit 0 (success), 3 (paused/blocked), or 6 (error).

## Official sources

- [Interaction and permissions](https://www.kimi.com/code/docs/en/kimi-code-cli/usage/interaction.html) · [Configuration files (permission rules)](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/config-files.html) · [Tools reference](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/tools.html) · [Hooks](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/hooks.html)
