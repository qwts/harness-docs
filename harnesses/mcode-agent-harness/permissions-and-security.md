# Permissions and security

**Verified:** 2026-11-15

## Permission modes

TUI picker: Ask (“Request interactive confirmation for tool actions”), Auto (“Classify routine actions and ask when risk is high”), Full access (“Skip normal permission checks”), plus Off (`permissionMode: off`) — switched with `Alt+M` or `/permission ask|auto|full`, inspected with `/permission status`. Headless uses policy names: `--permission smart|full|off` (default `smart`), and “`smart` corresponds to TUI Auto.” “Headless does not support `ask` because it has no human approval surface; use the TUI or ACP when a person must approve an action.” The `config.yaml` values are `default`, `auto`, `bypassPermissions`, `off`.

Plan Mode is orthogonal: “Plan Mode controls whether the agent plans before execution. Permission mode controls how tool actions are handled. They are independent settings,” and “An active plan does not change the permission policy for the execution phase.” The desktop permissions page lists what can require approval: “Reading files outside the workspace”, “Editing or deleting files”, “Running commands”, “Using tools with external side effects”, and “Handling actions requested through Remote Control or messaging channels.”

## Automation boundaries

CI baseline from the security page: explicit `--cwd` (“do not depend on the caller's current directory”), `--timeout` and `--max-steps` bounds, `--permission full` or `off` “only in an isolated workspace,” `--output-format json` or `stream-json` (“do not infer status from human-readable text”), and “Preserve stdout and stderr separately so diagnostics cannot corrupt machine output.” Headless “fails on pending questionnaires or permission requests instead of bypassing interaction.”

## Credential protection

“Provider API keys are read from environment variables” (`MCODE_PROVIDER_API_KEY`, or `--api-key-env`); “Do not put keys in a repository, `AGENTS.md`, prompts, screenshots, or CI logs. Login callback URLs can contain temporary credentials and must not be shared or committed. `mcode provider list` shows only managed-login or masked-key status.” Data-directory hygiene: “Use separate data directories on remote hosts, shared machines, and CI runners so accounts do not share sessions, logs, or provider configuration.”

## Protocol and capability boundaries

“ACP stdout is reserved for protocol messages; logs go to stderr.” Computer Use is the sharpest boundary: it is “available only in the interactive TUI after you explicitly select the official plugin. Headless runs, ACP, scheduled tasks, and subtasks cannot control the computer. Browser and other desktop-app capabilities are not available in the CLI.” The Computer Use page adds: macOS and Windows only (“Linux” is listed under Not Supported), the TUI “must run in a graphical login session,” and concurrency is exclusive — “Only one of them can use Computer Use at a time; the other reports that the computer is busy until the current action ends.” Install path: `mcode plugin add computer-use@official` + `mcode plugin enable computer-use@official`, then `/computer-use status` and (macOS) `/computer-use permissions` — “Enabling the plugin does not request system permissions on its own; run the permissions command separately.” `--computer-use` at launch selects it for the session, requires a non-empty task, “exits with an error” if plugin/permissions are missing, and “does not install the plugin, request system permissions, or change global defaults.” The desktop variant (macOS 12+/Windows) shows a “blue glow appears along the edges of every display” with a “MiniMax Code is using your computer” bar at the top of the screen, and its “Enable Computer Use in conversations by default” covers “local interactive conversations only. Scheduled tasks, subtasks, IM channel conversations, and conversations on SSH dev machines or in the cloud do not get Computer Use from it.”

## Data directory

Default `~/.minimax` (the desktop troubleshooting page also references a sibling `.mavis` directory when diagnosing service startup); `MINIMAX_DATA_DIR` relocates it, `MAVIS_DATA_DIR` is the compatibility fallback. Proxy variables are honored with localhost always bypassed.

## Official sources

- [Security and permissions](https://agent.minimax.io/docs/cli/security.md) · [Headless and CI](https://agent.minimax.io/docs/cli/automation.md) · [Computer Use (CLI)](https://agent.minimax.io/docs/cli/computer-use.md) · [Computer Use (desktop)](https://agent.minimax.io/docs/code/desktop/computer-use.md) · [Permissions and Safety](https://agent.minimax.io/docs/code/workflows/permissions.md) · [Troubleshooting](https://agent.minimax.io/docs/cli/faq.md)
