# Permissions, approvals, and execution profiles

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main)

## The permission model

“The Warp Agent CLI uses the same permission model as the Warp app: you choose how much approval each type of action needs.” “Every action the agent proposes, such as running a shell command, editing files, or calling an MCP tool, is checked against your active execution profile before it runs.” “Each action type is set independently” to one of three values:

- **`agent_decides`** — “The agent acts on its own when it's confident and asks when it's uncertain.”
- **`always_ask`** — “Every action of this type requires approval.”
- **`always_allow`** — “Actions of this type run without prompting.”

The app's Agent Profiles add a fourth level, **Never** (“Agent will not ever take the action (i.e. Create plans)”). Profile autonomy types in the app: apply code diffs, read files, create plans, execute commands, interact with running commands (via Full Terminal Use), and ask clarifying questions.

CLI defaults: “**Shell commands, file edits, file reads, MCP tool calls** - `agent_decides`. For file edits, `agent_decides` still prompts every time, so you always review a diff before it's applied.” Plus: “**Typing into a running command, launching child agents** - Always asks,” and “**Command denylist** - Sits on top of these values. Matching commands require approval even when `execute_commands` is `always_allow`.”

Diff caveat: “For **Apply code diffs**, `Agent decides` currently behaves the same as `Always ask` — the Agent always prompts you to review diffs before applying them. Only `Always allow` skips the review prompt.”

## Execution profiles (CLI)

“Profiles live under the `agents.execution_profiles` table, and the CLI always runs with the profile under the reserved `default` key”:

```toml
[agents.execution_profiles.default]
name = "Default"
execute_commands = "agent_decides"
apply_code_diffs = "agent_decides"
read_files = "agent_decides"
command_allowlist = ['cargo (build|check|test)(\s.*)?']
```

Fields: `execute_commands`, `apply_code_diffs`, `read_files`, `mcp_permissions`, `write_to_pty` (“Permission to type into running interactive commands. Also accepts `ask_on_first_write`”), `ask_user_question` (“`always_ask`, `ask_except_in_auto_approve`, or `never`”), `run_agents` (“`always_ask`, `always_allow`, or `never_allow`”), `command_allowlist` (“Regular expressions for commands that run without approval”), `command_denylist` (“Regular expressions for commands that require approval regardless of the other permission values. Auto-approve bypasses this list by default”), `directory_allowlist`, and model overrides such as `base_model`.

Isolation and fail-safety: “Profiles in the Warp Agent CLI are local to your machine. They never sync to the cloud, and they are separate from the Agent Profiles you configure in the Warp app. You can define additional profiles in the file, but the CLI currently always runs with `default`.” Validation: “The profile collection is validated as a whole. If any profile contains an invalid value, the CLI keeps the last valid configuration while it's running and falls back to the built-in default profile on the next launch, until the file is fixed.”

## Allowlist and denylist

“The Agent lets you define an allowlist of commands that run automatically without confirmation. It's empty by default, but users often add read-only commands such as” `which .*`, `ls(\s.*)?`, `grep(\s.*)?`, `find .*`, `echo(\s.*)?`. “Commands in the allowlist will always auto-execute, even if they are not read-only operations.”

“For safety, the Agent always prompts for confirmation before executing potentially risky commands.” The built-in default denylist “covers `rm`, `curl`, `wget`, `eval`, `ssh`, shells, and other risky command patterns. Omitting the field keeps the defaults. To deny additional commands, extend the generated list in your settings file rather than writing a short list from scratch.” Precedence: “The denylist takes precedence over both the allowlist and `Agent decides`: if a command matches the denylist, the Agent asks for permission even when that action type is set to **Always allow**.”

MCP permissions mirror this: “Use the MCP allowlist to give the Agent permission to call specific servers without asking”; “Use the MCP denylist to require approval before calling certain servers, even if they're also in the allowlist”; or set “decide.”

## Auto-approve and Run until completion

“Auto-approve gives the agent full autonomy. Proposed actions run immediately, without permission cards, until the task finishes or you turn it off. Toggle it with `/auto-approve` or `Ctrl+Shift+I`.” Scope: “Auto-approve is scoped to a single conversation, and new conversations start with it off. To start every new conversation in a session with auto-approve already on, launch with `warp --auto-approve`.” (The flag “applies only to the session you launch with it and doesn't change your saved settings.”) If `ask_user_question = "ask_except_in_auto_approve"`, “the agent also skips clarifying questions while auto-approve is on.”

The sharp edge: “With auto-approve on, the agent runs commands and applies file edits without review, including commands that match your own command denylist. Press `Ctrl+C` to stop the agent if it starts doing something you didn't intend.” Keep the denylist in force with:

```toml
[agents.warp_agent.other]
auto_approve_bypasses_command_denylist = false
```

“Denylist rules enforced by your team in the Admin Panel always require approval and are never bypassed, regardless of this setting.” The app calls the same behavior **Run until completion** — the purest form of YOLO mode — “the Agent proceeds without asking for confirmation, and by default it also runs commands that match your command denylist” — toggled off via **Allow auto-approve to bypass command denylist** in settings; all-permissions-**Always allow** is described as full autonomy (“YOLO mode”) with denylists still overriding.

## Approval UX

When an action needs approval, “the agent pauses and shows a permission card with the proposed command or file edits.” Beyond approve/reject: “Select **Other** to reply with guidance instead of running the action. The agent adjusts its approach based on what you type”; `E` edits a proposed command before approval (`Esc` exits without rejecting) or expands/collapses all diffs on a file-edits card. Keyboard: `Enter` confirms, `Esc` rejects or cancels, `Ctrl+Enter` approves “a request to type into a running command.”

## Full Terminal Use

“Full Terminal Use lets the Warp Agent operate directly inside interactive terminal applications like database shells, debuggers, text editors, and long-running servers. The agent can see the live terminal buffer, write commands, respond to prompts, and hand control back to you at any time.” You can ask the agent to start an interactive program or tag it into one you're already running (the **Use Agent** button or `Cmd+I`); “Warp then attaches the agent to the active PTY so it can see the current terminal buffer and propose actions inside the session.” In the CLI, terminal control keys take over running commands: `Ctrl+C` interrupts yours or takes control of the agent's, `Ctrl+G` hands control back.

## Official sources

- [Permissions and profiles (CLI)](https://docs.warp.dev/agents/cli/permissions-and-profiles) · [Profiles & Permissions](https://docs.warp.dev/agents/capabilities/agent-profiles-permissions) · [Full Terminal Use](https://docs.warp.dev/agents/capabilities/full-terminal-use) · [Agent questions](https://docs.warp.dev/agents/local-agents/interacting-with-agents/agent-questions) · [CLI reference](https://docs.warp.dev/agents/cli/reference)
