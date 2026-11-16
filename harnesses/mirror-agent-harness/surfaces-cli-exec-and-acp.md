# CLI surfaces, exec, and ACP

**Verified:** 2026-11-15

## Install and platforms

Install is `uv tool install reflection-mirror`, and “The package is named `reflection-mirror`; the command is `mirror`. Release wheels include the native agent and ripgrep, so no Rust build is required.” Platforms: “Mirror CLI runs on macOS 14 or later on Apple Silicon, and on Linux with glibc 2.28 or later on x86-64 or ARM64. Git or Jujutsu is needed for worktree management; you can start a session in a plain directory without either. Native Windows isn't supported.” Running from source needs uv, “Rust/Cargo, a C compiler, `protoc`, and ripgrep (`rg`)” plus Python 3.12+ via uv, launched with `./mirror-cli/mirror-dev`; “The Rust checkpoint tests also require Jujutsu (`jj`) on `PATH`.”

Updates: “Interactive installations can check for updates and ask before installing a newer release. … Use `MIRROR_SKIP_UPDATE=1 mirror` to bypass both checks and cached releases. … Run `mirror update` in an interactive terminal to check for an update without starting a session. It asks before installing and exits when finished. Updates require `uv` on your `PATH`. To upgrade the original tool installation, run `uv tool upgrade reflection-mirror`.” Launch flags `--update` (“Allow a longer startup update check”) and `--no-update` (“Skip the startup update check. A newer cached release can still be selected”) bound the check per launch.

## First run and sign-in

“The first time you start Mirror in a directory, it asks you to confirm that you trust that directory's files” (`Continue and trust this folder? [y/N]`), after a one-time “Using Mirror” notice. “Without an environment key or a saved key, Mirror CLI opens a **Connect to Reflection** screen with a link to create one. … Mirror CLI saves the key to `~/.mirror/reflection_api_key`, readable only by your user. … It doesn't check the key when it saves it; if the key is wrong, requests fail with `401`, and `/logout` removes the saved key so you can paste another.” “An environment key takes precedence over the saved key.”

## TUI shortcuts

| Key | Action |
|---|---|
| `Enter` | “Send the prompt. While the agent works, queue it for the agent.” |
| `Ctrl+J` | “Insert a new line. `Shift+Enter` also works in terminals that report it.” |
| `Escape` | “Stop the agent's current turn, including at a tool-approval prompt. Queued messages return to the prompt.” |
| `Ctrl+C` | “Dismiss a menu, clear a draft and save it to history, interrupt active work with an empty prompt, or quit when idle.” |
| `Ctrl+D` | Exit. |
| `Ctrl+R` | “Search prompt history.” |
| `Ctrl+B` | “Move running commands to the background so the agent can continue.” |
| `Down` | “Select a background task. Then `Up` and `Down` move, and `x` cancels it.” |
| `Tab` / `Shift+Tab` | “Switch to the next or previous workspace.” |
| `Ctrl+O` / `Ctrl+N` | Open the workspace list / “Choose a new workspace in the current directory or a new worktree.” |
| `F2` | “Switch between automatic and manual tool approval.” |
| `Ctrl+T` | “Change the transcript detail.” |
| `Ctrl+P` / `Ctrl+G` | Open the editor's file picker / “Edit the draft prompt in an external editor.” |
| `$` | “Browse skills.” |

`~/.mirror/keybindings.json` remaps actions (“Each entry replaces all the keys for one action, and an empty `keys` list disables the action”), with two locks: “`Escape` can only be bound to `InterruptRequested`. Projects can't change your shortcuts.”

## Slash commands

Conversations and workspaces: `/clear`, `/resume [id]`, `/rewind`, `/fork [name]`, `/compact`, `/worktree [name]`, `/folder <path>`, `/sandbox [name]`, `/workspaces`, `/rename <name>`, `/tag`, `/close`, `/drop`, `/status` (“Show the model, reasoning effort, conversation ID, working directory, and whether a turn is running”), `/usage`. Models and settings: `/model [model-or-provider]`, `/effort [level]`, `/login`, `/logout`, `/approvals`, `/connect [service]`, `/connections`, `/web-provider [you|tavily|firecrawl]`, `/disconnect [service]`, `/settings`, `/memory [status|import]`, `/extensions`, `/extensions-add`, `/extensions-update`, `/extensions-remove`, `/context`. Screen and session: `/view [mode]`, `/themes [theme-id]`, `/tips [enable|disable]`, `/editor`, `/help`, `/feedback`, `/quit`, `/exit`. “Commands marked Idle only are rejected while a turn is running.”

## Noninteractive exec

“Passing a prompt is shorthand for `mirror exec PROMPT`. Both run one new local session without the TUI and print the final answer. … Omit the prompt or use `-` to read it from stdin. Piped input alongside a prompt argument is appended in a `<stdin>` block.” Ground rules: “First open Mirror interactively in that directory to accept the usage and folder notices. Supply provider credentials through the environment or a saved login; `exec` rejects `--api-key`. Calls that require approval fail, so select `--approval-mode auto` only when you want automatic tool approval. `exec` starts a fresh session without extensions and doesn't support `--continue` or `--conversation-id`.” (There is no machine-readable output-format table like other harnesses — exec prints the final answer; `--log-level` tunes logging from `debug`/`info` (default)/`warning`/`error`/`critical`.)

## ACP and remote gateways

“Run `mirror acp` to serve the Agent Client Protocol on stdin/stdout for a client such as Zed or Omnigent. The client sends prompts and receives responses and tool calls. Approval requests go to the client, and cancellation interrupts the turn. Model and reasoning flags apply. Like `exec`, ACP uses environment credentials or a saved login, starts fresh sessions without extensions, and requires the directory's notices to have been accepted interactively.”

Remote operation exists but is opt-in: `--remote-endpoint <endpoint>` — “Configure an agent gateway supplied by your service operator. Remote gateways are disabled unless configured” — with `MIRROR_REMOTE_ENDPOINT` as the environment equivalent and `--remote-insecure` for “plaintext gRPC for a loopback remote endpoint.” Memory is likewise unavailable in remote sessions. There is no scheduler, daemon, or wakeup surface: “Quitting Mirror stops every agent. Nothing keeps running in the background after you exit.”

## Environment variables and files

Provider and web keys: `REFLECTION_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`, `GEMINI_API_KEY`, `YOU_API_KEY`, `TAVILY_API_KEY`, `FIRECRAWL_API_KEY`; endpoints `OPENAI_BASE_URL`, `ANTHROPIC_BASE_URL`, `OPENROUTER_BASE_URL`; editor `VISUAL`/`EDITOR` (“Defaults to `vi`”); `MIRROR_MODELS_CONFIG`, `MIRROR_MCP_DIR`, `MIRROR_EXTENSIONS_ENABLED`, `MIRROR_SKIP_UPDATE`, `MIRROR_CODEX_AUTH_PATH`, `MIRROR_REMOTE_ENDPOINT`, `MIRROR_STATE_DB`, and `MIRROR_CLI_DB` (“Path of the database holding prompt history and workspace settings. It must differ from `MIRROR_STATE_DB`”). Agent-run commands receive none of the API-key variables.

Notable launch options beyond those covered elsewhere: `--allow-foreign-workspace` (“Resume a conversation from another directory without asking”), `--showcase` (“Explore synthetic conversation history without a model or real tools”), `--max-iterations <count>` (“Cap the model and tool steps in a turn. Unlimited by default”), `--no-stream`, `--no-stream-usage`, `--no-strict-finish-reason`, `--scrollback-limit <lines>` (“Defaults to 1000; `0` means no limit”), `--flush-scrollback`, `--database <path>` (“Store conversations in this database instead of `~/.mirror/state.db`”), `--version`.

`~/.mirror/` layout: `state.db` (conversations), `cli.db` (prompt history and workspace settings), `config.json` (approval mode, trusted directories, extension settings, `web_provider`, analytics choice and install ID, `display_unmanaged_worktrees`), `tool_approvals.json`, `models.json`, `last_model.json`, `keybindings.json`, `reflection_api_key`, `AGENTS.md`, `skills/`, `extensions/`, `mcp/` (+ `credentials/`), `memory/` and `PINNED_MEMORIES.md`, `worktrees/`, `sandboxes/`, `logs/<conversation-id>/`, `analytics/`. “Reinstalling or upgrading Mirror doesn't change these files.” Repository-side reads, per directory from root to working directory: `AGENTS.md` (with `CLAUDE.md` fallback), `.agents/skills/` and `.claude/skills/`, and `.mirror/extensions/`.

## Official sources

- [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md) · [Coding-agents quickstart](https://developers.reflection.ai/coding-agents-quickstart.md) · [GitHub: reflection-oss/mirror](https://github.com/reflection-oss/mirror) · [Mirror CLI overview](https://developers.reflection.ai/mirror-overview.md)
