# Agents, sessions, and worktrees

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site main)

## Agents

“Agents are **configuration overrides** applied on top of the global config. They bundle a system prompt, a compaction prompt, an active model, an available tool set, and approval rules.”

Built-in agents: `ask` (“Asks for approval before running tools. Formerly named `default`.”), `accept-edits` (“**Default agent.** Auto-approves file edits in the working directory”), `smart-approve` (“Classifies each tool call, runs the ones it judges safe, and asks for approval before risky ones”), `auto-approve` (“Auto-approves all tool execution … can run arbitrary commands such as `rm -rf`”; disabled by admin `enable_auto_approve = false`), and `lean` (“Specialized agent for the Lean 4 proof assistant. Install with `/leanstall`”). There is **no built-in `plan` agent** — a missing custom `plan` falls back to `ask` with a warning.

Selection: `vibe --agent <name>`, `Shift+Tab` mid-session, or `default_agent` in config (“applies to both interactive and programmatic (`--prompt`) sessions”). `--auto-approve` / `--yolo` approve every tool call for one run and are ignored under `enable_auto_approve = false`.

**Custom agents** live in `~/.vibe/agents/` or `./.vibe/agents/` as `.toml` files with `agent_type`: `"agent"` (user-facing, selectable) or `"subagent"` (“**delegation-only**. Spawned by the agent when it delegates work. Not selectable by the user”). “The name `main` is reserved.” Fields include `display_name`, `description`, `safety` (`"safe"` / `"yolo"`), `active_model`, `system_prompt_id`, `disabled_tools`, and `[tools.<name>] permission = "always" | "ask" | "never"`. “The agent can also delegate work to **subagents**, for example to explore a large codebase without filling its own context. You can't select subagents with `--agent`.” Subagents are gated by `enable_subagents` (true) with `max_subagents` 32 and `max_concurrent_subagents` 32.

## Interactive and programmatic modes

“The CLI runs in **two modes**: *interactive* for chat and exploration, and *programmatic* for scripting and CI.” Interactive features: `@file` references with autocomplete (images too — “On macOS, you can also paste an image from the clipboard with `Ctrl+V` or `/paste-image`”; “Images work when the active model supports them or when a `vision_model` is configured”), `/` slash commands, `!` shell passthrough (“The agent sees the command and its output as context”), and `&` to “Send a prompt to a cloud session”.

Programmatic: `vibe --prompt …` uses your `default_agent`, “disables interactive tools such as `ask_user_question`”, and — since “Nobody can answer an approval prompt” — “Vibe denies every tool call that needs approval” unless `--auto-approve`/`--yolo` is passed. “**Programmatic mode never prompts for trust.** Use `--trust` to grant temporary trust when scripting Vibe.”

## Sessions

- **Resume/history**: `/resume` (alias `/continue`) — “Browse, resume, or delete saved sessions”; `-c`/`--resume` on launch; `/branch` (`/fork`) — “Fork the current conversation into a new resumable session”; `/rewind` (“Rewind to a previous message (or press Esc twice)”); `/rename`; `/clear` (`/new`); `/compact` — “Compact conversation history by summarizing”; `/retry`; `/copy`; `/context` — “Show what fills the context window … and estimated tokens per category”; `/pr` — “Show this session's pull requests … press `c` to open the first failing CI check, or `f` to draft a prompt that fixes what blocks the PR”; `/loop` — “Schedule a recurring prompt in plain language”; `/status`, `/whoami`, `/log`, `/log-level`, `/debug`, `/data-retention`, `/release-notes` (`/whatsnew`), `/reload`, `/theme`, `/voice`, `/proxy-setup`.
- **Scheduled prompts**: `/loop` (“`/loop every two minutes check the build`”) — “Ask the agent to list or cancel schedules.”
- **Background processes**: `enable_background_processes = true` lets “the agent run commands in the background”; `false` forces foreground completion.

## Worktrees

“Use `--worktree` to run a session in its own git worktree, so the agent's changes stay apart from your current checkout”; “Vibe creates the worktree under `~/.vibe/worktrees/` on a branch named `my-feature`, or reuses it if it already exists. Without a name, Vibe names the worktree after your prompt.” Cleanup: “When an interactive session ends, Vibe removes a worktree it created if it holds no changes, and asks you otherwise. Programmatic runs leave the worktree in place.” “On macOS, Vibe creates new worktrees of a trusted repository from a template checkout … sets `extensions.worktreeConfig` in the repository's `.git/config`” (disable with `worktree_copy_on_write = false`; `worktree_limit` 15 keeps recent worktrees). “`--worktree` trusts the worktree for that run, so you don't need `--trust`. Combine it with `-c` or `--resume` to resume a session started in that worktree.”

## Terminal and shortcuts

“The Vibe CLI runs in any modern terminal emulator, but terminals don't all support the same features” (dedicated terminal-support page). Key bindings include `Shift+Tab` (cycle agents), `Ctrl+R` (prompt history search), `Ctrl+G` (external editor), `Ctrl+O` (toggle tool output), `Esc Esc` (rewind), `Cmd+\`/`Alt+\` (todo list panel), and exit semantics via `Ctrl+C`/`Ctrl+D` (“`ask_confirmation_on_exit` … Asks for a second `Ctrl+D` before quitting”).

## Official sources

- [Agents](https://docs.mistral.ai/vibe/code/cli/agents) · [Work with the CLI](https://docs.mistral.ai/vibe/code/cli/work-with-cli) · [Commands and shortcuts](https://docs.mistral.ai/vibe/code/cli/commands-shortcuts) · [Terminal support](https://docs.mistral.ai/vibe/code/cli/terminal-support) · [Hooks](https://docs.mistral.ai/vibe/code/cli/hooks)
