# Permissions and execution modes

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; app v3.18.2)

## The four execution modes

“Execution modes control how ZCode Agent asks for confirmation before editing files or running commands.” “There are four execution modes. Press Shift + Tab while the chat input is focused to cycle through them”:

| Mode | Behavior |
|---|---|
| Ask before changes | “Asks for confirmation before every file edit or command; this is the default” |
| Edit automatically | “Applies file edits automatically while commands still require confirmation” |
| Plan mode | “Reviews the code and proposes a plan first, then starts editing after confirmation” |
| Full access | “Reduces interruptions and lets the Agent proceed more continuously” |

“ZCode Agent is ZCode's first-party Agent. It is tuned closely for the GLM 5.3 model family”; the docs' guidance: “For critical files, shell commands, or broad changes, start with Plan mode or Ask before changes. For routine edits or clearly scoped tasks, Edit automatically or Full access can reduce interruptions.” The v3.18.2 changelog adds “You can now select the permission mode directly during authorization, with the option to remember Full Access.”

## What gets surfaced

“Before allowing execution, review what the Agent is about to do. Commands, file changes, network calls, and script execution are surfaced in the task. When a high-permission or fully automatic mode is active, ZCode keeps the risk state visible in the toolbar.” Typical gated scenarios: “Running third-party scripts … Network requests: external API access through curl or similar tools … File changes … ZCode shows a file-change summary with open and undo actions … System-level commands.”

## The confirmation workflow

“Trigger confirmation: when an Agent sends a permission-gated request, the current task pauses and the composer is blocked so the next action cannot be queued accidentally.” “ZCode displays the exact command, file change, or tool action the Agent plans to run,” the Agent “continues only after you approve the request; rejecting it stops the current operation or returns the task to an adjustable state,” and “permission requests are scoped to the task. If you switch away and come back, the pending request is still there, and the sidebar can show the task as waiting for confirmation.”

Decision options: **Allow** (“Authorize only this one action”), **Always Allow** (“Authorize this and future actions of the same type without asking again”), **Reject**, **Always Reject** (“Block this and future actions of the same type”). “In some scenarios, finer-grained options such as Allow for this session or Always allow for this project are also available.”

## Agent questions

“Beyond permission requests, the Agent sometimes just asks you a question — “approach A or approach B?”” These wait five minutes by default, with a countdown appearing in the panel as the time runs low. “If you don't answer, the Agent carries on using its own judgment, and that step is marked “Unanswered, auto-continued” in the conversation history.” “Any sign of activity — moving the pointer onto the panel, starting to choose, clicking the countdown, or clicking “stop timer” in the sidebar — stops the countdown permanently”. Disable via “Settings → General → Automatically continue questions; every question will then wait indefinitely. Note that the switch isn't retroactive.” Scope: “The countdown applies only to these ordinary questions. Permission requests and plan approvals always wait and are unaffected.”

## Hooks (policy enforcement around the agent)

Hooks “run actions automatically at specific events — inject team constraints when a session starts, run safety checks before a tool call, or validate output when the model is about to finish. They are built for plugin developers and project maintainers.” “A hook is essentially a local subprocess protocol: ZCode writes one line of JSON to the process's stdin, and the process replies through its exit code and stdout JSON. Hooks never receive internal objects that can call the ZCode model directly.” “Security note: hooks execute local code. Review the source, hooks/hooks.json, and scripts of any third-party plugin before enabling it.”

Event flow: “New session → SessionStart”; “User submits → UserPromptSubmit → main model”; “Main model requests a tool → PreToolUse → PermissionRequest (when confirmation is needed) → tool runs”; “Tool succeeds → PostToolUse; tool fails → PostToolUseFailure”; “Main model about to finish → Stop”. Matchers: SessionStart “Matches source; common values are startup / clear / compact”; UserPromptSubmit “cannot rewrite the original prompt”; PreToolUse “Allow, ask, or deny the tool; may fully replace the tool input, which is then re-validated against the schema”; PermissionRequest “Fires only when the permission outcome would prompt the user; can allow, deny, update the input, or update permission rules”; PostToolUse “cannot replace the tool output”; PostToolUseFailure appends recovery suggestions; Stop “returning block lets the main-model loop continue, at most 3 times in a row.”

Sources: “`~/.zcode/cli/config.json` … The file must set `hooks.enabled: true`”; plugin `hooks/hooks.json` (“Auto-discovered at the standard location, follows the plugin's enable state”); `.agents/settings.json` / `.claude/settings.json` are “Read-only display, not executed; import explicitly into .zcode from the settings page.” Project-level hooks are not executed: “any hooks configuration in `<workspace>/.zcode/config.json` or `<workspace>/zcode.json` is ignored as a whole — regardless of hooks.enabled — and logged as `config_project_hooks_ignored` … To share hooks across a team, distribute them via a plugin … or have each member configure user-level hooks.”

## Official sources

- [Safety Confirmation](https://zcode.z.ai/en/docs/safety-confirm) · [Hooks](https://zcode.z.ai/en/docs/hooks) · [ZCode Agent](https://zcode.z.ai/en/docs/agents)
