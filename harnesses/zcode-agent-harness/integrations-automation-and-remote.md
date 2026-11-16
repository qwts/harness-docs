# MCP, plugins, automations, and remote surfaces

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; app v3.18.2)

## MCP servers

“MCP (Model Context Protocol) connects external capabilities such as file systems, browser automation, memory, and databases to Agents. ZCode manages the MCP server configuration used by ZCode Agent in one place.” The list groups servers into “Configured MCP servers” (manual, “edit, delete, enable, or disable”) and “Plugin MCP servers” (“installed together with plugins, managed by the corresponding plugin”).

- **Creation** — Settings → MCP Servers → New MCP Server: “Choose a Scope: User (available in all workspaces) or Workspace (current project only)”; type “stdio (SSE and HTTP remote servers are also supported)”; command + args (e.g. `npx`, `-y @modelcontextprotocol/server-memory`); env vars; or **Full configuration mode** — “ZCode accepts both `{"server-name": {...}}` and `{"mcpServers": {...}}` style structures.”
- **OAuth** — “Some remote MCP services — Figma's official MCP among them — don't accept a static key and ask you to sign in instead. Enable OAuth for the service in its configuration and ZCode takes care of the rest.” Clicking “Open authorization … opens the authorization page in your system browser … ZCode never opens a browser window on its own.” “This applies to HTTP and SSE remote services only — local servers still get their credentials through environment variables. If authorization doesn't come through in time when a session starts, ZCode marks that one server as failed and carries on rather than holding up the session.”
- **Import** — “If you already configured MCP servers in external Agents such as Claude Code, Codex CLI, or OpenCode … click the Im[port]” button on the MCP Servers page.

## Plugins

“Plugins extend what ZCode can do. A single plugin can bundle skills, commands, subagents, and MCP servers” — plus hooks. “When you enable a plugin, its runnable components — skills, commands, and MCP servers — are registered into the current workspace; disabling it disables them together.”

Store: Settings → Plugins lands in “the plugin store,” split into **Public** (“the catalog ZCode curates … served from GitHub, so the catalog, plugin details, and installs may fail when GitHub is unreachable”) and **Personal** (your marketplaces; “ZCode preloads the Claude Code marketplace here, so you can browse and install from it without adding anything”). Add marketplaces from “A GitHub repository (e.g. owner/repo or its URL)”, “A Git URL”, or “A local marketplace file or directory”. “The plugin page requires an open workspace.” “Newly installed plugins are enabled by default.” Hook distribution: plugin `hooks/hooks.json` is “Auto-discovered at the standard location, follows the plugin's enable state.”

## Automations (scheduled tasks)

“Automations let ZCode Agent run on the cadence you set and leave the results in a session for you to review.” Created from a form (title, schedule, instructions, and “Project / permission / model / thought level” toolbar), from a chat session (“A task created this way is bound to the current session: every trigger delivers its results back to that same session”), or from four read-only templates (“Weekday standup digest … Daily risk scan … Friday release brief … Wednesday doc sync check … read-only analysis, only verifiable facts from the repository … no speculation and no code changes”).

- **Schedule rules** — “Hourly / Daily / Every weekday / Weekly / Monthly / Custom”; Custom adds “Repeat unit … Interval … Specific timing … By date or by weekday” (e.g. “the third Monday”).
- **Execution locality** — “Your computer must be awake. Scheduled tasks run on your machine, so nothing triggers while it's asleep or the app is closed, and a missed one-time run is recorded as skipped rather than replayed. For tasks scheduled overnight, turn on Keep awake on the Automations page.” “Everything is computed in your computer's local timezone.”
- **Task config** — “Project: which project directory it runs in. Only local projects currently open in the window are selectable, and this can't be changed after creation”; “Permission: the same four execution modes as a regular session … The icon stays orange when Full access is selected”; model and thought level “leave empty to follow the project default.”
- **Lifetime** — “Scheduled tasks repeat indefinitely by default” … pick a date under Ends … “There's no “run at most N times” field in the form. When you do need a fixed count, ask the Agent in a conversation to create the scheduled task and say how many times it should run.” “Run Now … doesn't disturb the schedule.”

## Idle-time tasks

“Not every task needs to start right away … it joins a queue that runs during periods of spare computing capacity. The run is free and doesn't consume your subscription plan quota.” “An idle-time task has no schedule, runs once, and starts whenever the queue reaches it.” Prerequisites: “Idle-time tasks require an active Coding Plan subscription. Personal Coding Plans from both ZAI and BigModel work; Start Plan and plain API key mode are not supported. Support for team plans is still rolling out.” “Rolling out gradually. Idle-time tasks are being released to subscribers in batches.” Same locality rule: “Your computer needs to stay awake … Nothing progresses while it's asleep, but nothing is lost either — it keeps queueing once you're back.” Instructions “Since it runs unattended, make them complete and self-contained”; “Only local projects” are selectable.

## Remote Control (phone)

“Remote Control temporarily opens the current ZCode desktop window to your phone. Once connected, you can browse and switch between the workspaces, tasks, and sessions already open in that window, and keep sending instructions to the Agent.” Boundary: “The phone is a control surface: it doesn't sync code and never creates a runtime of its own. Every instruction you send is carried out by whichever machine that workspace was already connected to on the desktop.” Connect via “the phone icon in the lower-left sidebar” — “a QR code and a link … Scan the code with your phone, or copy the link and open it in a mobile browser.” Scope: “your phone reaches the entire desktop window, not just the workspace you happened to be looking at when you scanned.” “Only one phone page can be connected at a time.”

## Bot Channel (WeChat / Feishu)

“Bot Channel connects external chat tools to ZCode. After setup, you can open the current workspace from WeChat or Feishu, check task progress, and keep sending instructions to ZCode Agent.” Channels: “WeChat: scan to sign in and bind automatically”; “Feishu: scan to create an app, then bind through messages” (time-limited “`/bind C67356`”-style command); “DingTalk, Discord, and WeCom support will be completed in later versions.” Bot settings: “enable or disable a bot, check its connection status, set reply granularity, and limit which workspaces it can access.”

## Remote Development

“ZCode remote development lets you run a workspace inside a remote host, a WSL distribution, or a local container. After the connection succeeds, file reads, terminal commands, Git operations, and ZCode Agent execution happen in the target environment; the desktop client still handles your account, model configuration, task entry points, and interface.” “The target needs a POSIX shell. Connecting to a native Windows remote host isn't supported. If you want the Linux environment on your own Windows machine, choose WSL.” “ZCode currently shows SSH remote host and Docker local container options” from the Open Workspace page.

## External triggering: what exists and what doesn't

Verified from the docs: automation entry points are the desktop UI itself (form/chat/templates), phone Remote Control, and Bot Channel — all bound to a running desktop app on your machine. “Nothing triggers while it's asleep or the app is closed.” The docs document **no CLI, no headless mode, no public API/webhook/server-side scheduler** for ZCode; the only non-desktop runtimes are the optional SSH/WSL/Docker remote workspaces, which still execute through the desktop client. This is a verified absence, not an invented gap.

## Official sources

- [MCP](https://zcode.z.ai/en/docs/mcp-services) · [Plugin](https://zcode.z.ai/en/docs/plugin) · [Automations](https://zcode.z.ai/en/docs/automations) · [Idle-time Task](https://zcode.z.ai/en/docs/idle-time-tasks) · [Remote Control](https://zcode.z.ai/en/docs/remote-control) · [Bot Channel](https://zcode.z.ai/en/docs/bot-channel) · [Remote Development](https://zcode.z.ai/en/docs/remote-development) · [Hooks](https://zcode.z.ai/en/docs/hooks)
