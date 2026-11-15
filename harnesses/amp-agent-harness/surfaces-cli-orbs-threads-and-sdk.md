# Surfaces: CLI, Orbs, threads, and SDK

**Verified:** 2026-11-15

## CLI: interactive and execute mode

Bare `amp` runs in interactive mode; piped input becomes the first user message. Execute mode is the headless form: `amp -x "prompt"` “sends the message provided to -x to the agent, waits until the agent ends its turn, prints its final message, and exits” — and redirecting stdout turns execute mode on automatically. `amp -ox "…"` runs the same prompt in an orb on Amp’s servers; `--orb-size` and `--executor local|orb|runner:<id>` choose where threads run. Other execute-mode flags: `--mcp-config` (inline JSON), `--attach <path>` (repeatable), `--fast` (Fast mode for one invocation; new threads otherwise start in Standard mode), `--timings` (report to stderr), and `--plugin-ready-timeout` (bare waits up to 10 s, max 300, `0` disables — without it, `agent.start`/`agent.end` lifecycle events can be skipped).

The CLI supports macOS, Linux, and Windows through WSL. Accounts are per-server (`amp login`, `amp account list/switch`, `amp logout`); updates install in the background; shell completions come from `amp completions bash|zsh|fish|pwsh`; images paste with Ctrl+V. `ide connect` (Ctrl+O) links the CLI session to VS Code, Cursor, Windsurf, Zed, or Neovim (plugin required).

## Runners and cross-client access

A runner is an Amp instance on “a machine you already have” that runs remotely created threads: `amp --no-tui --runner-id <id>` waits for and runs threads in the current directory. Runners can serve extra directories (`--dir`, repeatable) or discover git checkouts (`--discover-dirs`, up to two levels deep by default, capped at 500), and a long-running runner updates itself hourly, restarting when idle at most once every 12 hours. Threads on a runner “run as the OS user that started the runner” and “can read and change that user’s files and use its credentials”; `--share` warns that “workspace members will be able to run code on the machine as you,” and admins can disable runner sharing workspace-wide. Cloud-managed Secrets & Env Vars (`--amp-env`) merge with precedence personal > project > workspace > runner process.

Cross-client access is “enabled by default” at the workspace level: a running CLI thread can be continued from ampcode.com on desktop or mobile. Terminal-level control additionally needs `amp --remote-control-terminal` (or `AMP_REMOTE_CONTROL_TERMINAL=1`); an explicit flag beats the environment variable. Orb threads are not remote-controlled (they are already sandboxed), and other members’ threads stay read-only. Admins can disable the feature and require passkey auth for all members.

## Orbs

“Orbs are cloud sandboxes where Amp agents work without using your computer.” Every orb thread gets “a fresh, isolated environment with your code, plugins, development tools, and the context of the thread that created it.” Orbs pause a few minutes after the agent finishes and interaction stops; “a sleeping orb costs nothing, even if it sleeps for weeks,” and waking restores files, services, and conversation. The terminal tab shares the agent’s filesystem in a tmux session, file uploads reach 512 MB each (Files pane or terminal), `amp sync <thread>` mirrors orb changes into a local checkout (or `--apply` for a one-time copy), and each user can start a burst of 20 metered orbs before new starts queue at one per five minutes.

## Threads

A thread — “one conversation with the agent” — is “the unit of work in Amp” and has a URL (`ampcode.com/threads/T-…`) that opens in every client. Messages sent while the agent works are steered (applied after the current step); ⌘/Ctrl+Enter queues them until the agent completely finishes. Archiving removes a thread from active lists while the URL keeps working; snoozing “does not stop the agent, and it does not pause the thread’s automation.” Export is Markdown (append `.md`) or full JSON for the creator (`amp threads markdown` / `amp threads export`). Sharing levels run Private → Workspace → Group (Enterprise) → Unlisted.

## Puck and companion surfaces

Puck is “your personal Amp assistant that helps you manage everything you do in Amp: start agents, monitor them, send messages to them, create projects” — reachable in the web app (Ctrl+/), the TUI command palette (`puck: open`), mobile sidebars, and Slack via `@Amp`. External MCP clients (Claude, ChatGPT) connect at `https://ampcode.com/mcp` with `puck`, `new_puck_conversation`, and `orbs` tools. Puck has no orb: “it can read a skill’s instructions and files but cannot run its scripts. Plugins do not load in Puck.” Spaces, Workspaces, Multiplayer (co-editing threads; unavailable on anyone-with-link threads), and Huddle are documented companion surfaces.

## SDK

`@ampcode/sdk` “allows you to programmatically use the Amp agent in your TypeScript or Python programs,” authenticated with `AMP_API_KEY` or a local `amp login`. The core call is `execute()`, streaming `system`, `assistant`, and `result` messages; options include `continue` (most recent thread or by `T-…` ID), `title` and `labels` (creation only), `cwd`, `mode` (`low`, `medium` default, `high`, `ultra`), `model` with `effort`, `executor` (orb or runner), `visibility`, `settingsFile`, `skills`, `mcpConfig`, and an `AbortSignal`. The installed SDK release pins a minimum Amp CLI version.

## Official sources

- [CLI](https://ampcode.com/docs/cli)
- [Execute mode](https://ampcode.com/docs/cli/execute-mode)
- [Runners](https://ampcode.com/docs/cli/runners)
- [Cross-client access](https://ampcode.com/docs/cli/remote-control)
- [Orbs](https://ampcode.com/docs/orbs)
- [Threads](https://ampcode.com/docs/threads)
- [Puck](https://ampcode.com/docs/puck)
- [SDK](https://ampcode.com/docs/sdk)
- [Apps](https://ampcode.com/docs/apps)
