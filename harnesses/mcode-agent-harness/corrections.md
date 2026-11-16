# MiniMax Code corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| `mcode` is the MiniMax Code desktop app. | ❌ FALSE | The CLI “does not depend on Electron or desktop IPC”; the desktop app is a separate install (macOS 12+/Windows 10+). Both share the MiniMax account, and WebUI shares `~/.minimax` with the CLI. |
| The CLI runs on any Linux, including Alpine. | ❌ FALSE | “Alpine and other musl-based Linux distributions are not currently supported by the installer.” |
| Headless `mcode exec` can run with `--permission ask`. | ❌ FALSE | “Headless does not support `ask` because it has no human approval surface; use the TUI or ACP when a person must approve an action.” |
| `--permission smart` equals TUI Ask mode. | ❌ FALSE | “`smart` corresponds to TUI Auto.” |
| Enabling Plan Mode changes what tools can run. | ❌ FALSE | “Plan Mode controls whether the agent plans before execution. Permission mode controls how tool actions are handled. They are independent settings,” and “An active plan does not change the permission policy for the execution phase.” |
| Full access also auto-approves data-modifying tools in `/btw` side conversations. | ❌ FALSE | “The side conversation retains the current tool permissions, but tools that can modify data always require your explicit confirmation, including in Full access mode.” |
| API keys are stored in `config.yaml`. | ❌ FALSE | “Provider API keys are read from environment variables” (`MCODE_PROVIDER_API_KEY`, or `--api-key-env`); `provider list` shows “only managed-login or masked-key status.” |
| The default API-key variable is `MINIMAX_API_KEY`. | ❌ FALSE | “The default variable is `MCODE_PROVIDER_API_KEY`.” |
| Login callback URLs are harmless to share. | ❌ FALSE | “Login callback URLs can contain temporary credentials and must not be shared or committed.” |
| `--resume` and `--continue` can combine. | ❌ FALSE | “`--session`, `--continue`, and the compatibility alias `--resume` cannot be combined.” |
| `mcode exec --effort` is saved with the session. | ❌ FALSE | “The override is not saved to the session. Resuming later with `--session` or `--continue` and no `--effort` uses the session's original level.” |
| `--model provider/model#high` replaces `--effort high`. | ❌ FALSE | “`#variant` is part of a model identity: `--model provider/model#high` does not replace `--effort high`.” |
| Headless accepts unlimited attachments. | ⚠️ PARTIAL | The automation page says “A run accepts up to 10 files and 100 MB total”; the 0.7.7 changelog raises the per-message attachment limit to 20 in TUI/exec/ACP with the 100 MiB total unchanged — the prose pages lag the changelog. |
| A failed `exec` run exits with code 1. | ❌ FALSE | The stable set is 0/2/3/4/6/7/70/130/141 — “Check both the process exit code and the JSON `status`.” |
| Diagnostics and results share stdout. | ❌ FALSE | “Task results go to stdout and diagnostics go to stderr, so stdout can be passed safely to `jq`, CI artifacts, or another process.” |
| Headless waits for pending permission prompts. | ❌ FALSE | “`mcode exec` fails on pending questionnaires or permission requests instead of bypassing interaction,” returning an interaction error. |
| Plugin packages may ship npm lifecycle scripts. | ❌ FALSE | “Symlinks, hard links, Git LFS pointers, Git submodules, package installation lifecycle scripts (`preinstall`, `install`, or `postinstall` in `package.json`), platform-specific native binaries, and files that depend on the executable bit are not accepted.” |
| Packaged plugin MCP supports the `http` transport alias. | ❌ FALSE | “`stdio`, `streamable-http`, and `sse` are supported. The `http` alias is not supported” for packaged plugin MCP. |
| `stdio.command` may point at a script path. | ❌ FALSE | “`stdio.command` may contain only the name of an interpreter or executable available on PATH; it must not contain a path. Reference scripts in the package through relative `args`.” |
| Plugins can bundle OAuth tokens for convenience. | ❌ FALSE | “Do not include OAuth credentials, access tokens, refresh tokens, client secrets, or API keys in a Plugin” — “Capabilities that require user identity must first be integrated through App/Connector.” |
| Plugin manifest supports hooks today. | ❌ FALSE | “The current third-party intake does not accept the `hooks` manifest field or Hook JSON files.” |
| A published plugin is visible and installed by default. | ❌ FALSE | A published name “describes the package distribution result, not its current Marketplace visibility,” and “A newly published name is listed by default; this does not mean it is automatically installed.” |
| Computer Use works in `mcode exec` and ACP. | ❌ FALSE | It is “available only in the interactive TUI after you explicitly select the official plugin. Headless runs, ACP, scheduled tasks, and subtasks cannot control the computer.” |
| Computer Use works on Linux. | ❌ FALSE | It “works on macOS and Windows”; Linux is listed under Not Supported. |
| Installing the Computer Use plugin also grants macOS permissions. | ❌ FALSE | “Enabling the plugin does not request system permissions on its own; run the permissions command separately.” |
| Desktop app and CLI can drive Computer Use simultaneously. | ❌ FALSE | “Only one of them can use Computer Use at a time; the other reports that the computer is busy until the current action ends.” |
| The desktop “Enable Computer Use in conversations by default” applies everywhere. | ❌ FALSE | “The default covers local interactive conversations only. Scheduled tasks, subtasks, IM channel conversations, and conversations on SSH dev machines or in the cloud do not get Computer Use from it.” |
| Browser, desktop panels, and desktop shortcuts are CLI features. | ❌ FALSE | “Browser and other desktop-app capabilities are not available in the CLI” — they belong to the desktop app (and WebUI's own panels). |
| The status line shows context capacity and remaining percentage together by default. | ❌ FALSE | From 0.4.12 the default “switches to the remaining percentage, such as `Context 80% left`, without showing both indicators”; both can be enabled only as explicitly configured items. |
| Gemini base URLs must include `/v1beta`. | ⚠️ PARTIAL | Only if you want a different path — “a Base URL with only the host gets `/v1beta` appended automatically.” |
| The desktop app runs on Linux. | ❌ FALSE | macOS 12 Monterey+ and Windows 10+ only. |
| Homebrew install works on Windows or defaults to the global build. | ❌ FALSE | “Homebrew installs are for macOS only,” and “Without `--language`, the Mainland China build is installed.” |
| Goal mode runs unattended without bounds. | ❌ FALSE | “Goal is not unrestricted background automation. It remains subject to the task scope, user control, available budget, and evidence that can actually be verified,” and it stops when achieved, blocked, or paused. |
| In the desktop app, typing `/goal` starts Goal mode. | ❌ FALSE | “Typing `/goal` directly in the input box does not enter Goal mode. Select **Goal** from the `/` command menu or the `+` menu first.” (The CLI's `/goal` slash command is separate.) |
| Scheduled tasks run while the machine sleeps. | ❌ FALSE | “Scheduled tasks depend on the desktop runtime. If the computer is asleep or the app is not running, tasks may not run on time.” |
| All messaging integrations are available in every locale. | ❌ FALSE | “Chinese / Mainland China” gets “WeChat, Feishu”; “English / global” gets “Telegram, WeChat, Lark.” |
| Cloud handoff files have no size limit and sync continuously. | ❌ FALSE | “Each handoff file and the total selected files must be no larger than 500 MB,” and “Local and cloud project files do not continuously sync. Later local edits are not uploaded automatically, and cloud changes are applied to the original workspace only after you confirm.” |
| WebUI is a first-party MiniMax product. | ❌ FALSE | “WebUI is a community project, developed and maintained by” Jia Xiaoyuan (@Fectivnfy112357) (mcode-webui/minimax-code, first-party code MIT), with its own `mcode-webui` launcher and npm package. |
| WebUI supports desktop-cloud task handoff. | ❌ FALSE | Device-to-cloud task handoff is “Not supported” — “Cloud sessions and handoff are desktop features.” |
| The SDK retries failed turns automatically. | ❌ FALSE | “The SDK never retries automatically, to avoid replaying writes with unknown outcomes.” |
| SDK permission policies map to Codex OS sandbox modes. | ❌ FALSE | “MiniMax Code's `smart` / `full` / `off` policies are not equivalent to Codex OS sandbox modes, and the SDK does not map between them.” |
| The SDK can drive multiple Turns concurrently on one Thread. | ❌ FALSE | “A Thread runs one Turn at a time. A concurrent call throws `ThreadBusyError`; Turns are not queued.” |
| The SDK resolves the CLI from the working directory by default. | ❌ FALSE | “`resolveFromCwd` defaults to `false`. Set it to `true` to also search the host working directory after the SDK-local package and before `PATH`. Leave it off for untrusted checkouts.” |
| A memory can be enabled for an ongoing conversation. | ❌ FALSE | Desktop `/memory`'s “Use memory” option: “Set this before the conversation starts; it cannot be changed afterward” (only “Generate memory” can change mid-conversation). |
| Turning memory off deletes saved memories. | ❌ FALSE | “Turning memory off does not delete saved memories.” |
| Skills are specified on their own docs page. | ⚠️ PARTIAL | The dedicated Skills page is currently an empty stub; the operative specification lives in the Plugin guide (`skills/<skill-name>/SKILL.md`, frontmatter `name` matching the directory and non-empty `description`, runtime exposure as `<plugin-name>:<skill-name>`). |
| The proxy configuration can cover localhost services. | ❌ FALSE | “The CLI always bypasses `localhost`, `127.0.0.1`, and `::1` for login callbacks and local services.” |
| ACP permission changes write through to the shared config. | ⚠️ PARTIAL | Pre-0.7.x they did — “Fixed ACP clients changing the permission mode in the shared `config.yaml`, affecting Desktop and other Runtimes. Permission mode changes now apply only to the current ACP process.” |
