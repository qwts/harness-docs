# Memory, sessions, subagents, and runtime features

**Verified:** 2026-11-15

## Instructions: AGENTS.md and friends

“Project rules are Markdown files that Grok loads into context for every session in a directory tree.” Discovery order: “1. Global rules in `~/.grok/` 2. Every directory from the repo root down to the working directory (or only the working directory outside a git repo),” with “deeper files taking precedence on conflicts.” Per directory Grok reads “any of `AGENTS.md`, `Agents.md`, `AGENT.md`, `CLAUDE.md`, `Claude.md`, and `CLAUDE.local.md`, plus every `*.md` file in a `.grok/rules/` directory (`.claude/rules/` and `.cursor/rules/` are read for compatibility).” Gitignored files are skipped (“which keeps personal overrides like `CLAUDE.local.md` out of shared context”); “Files are loaded in full, with no size cap.” A nested `AGENTS.md` scopes to its subtree. Session rules: `--rules` “appends the text to the system prompt”; `--system-prompt-override` “replace[s] the system prompt entirely.” `GROK_RESPECT_GITIGNORE` forces gitignore filtering for search/read tools.

## Sessions

“Grok saves every conversation to disk automatically — prompts, responses, tool calls, and file snapshots — under `~/.grok/sessions/`, keyed by working directory. Sessions work the same in the TUI, headless mode, and over ACP.” Resume with `grok --resume <session-id>`, `--resume` (most recent for the directory), or `-c`; headless automation reads the ID back: `grok -p "..." --output-format json | jq -r '.sessionId'`. “`-s, --session-id` names a new session with a UUID you supply; it does not resume existing ones. To branch a resumed session instead of continuing it, add `--fork-session`.”

- **Fork:** “`/fork [directive]` branches the current session into a peer that starts from a copy of the conversation,” with `--worktree`/`--no-worktree` isolation.
- **Rewind:** “`/rewind` (or `Esc Esc` while idle) lists a rewind point per prompt. Selecting one restores all files to their state at that point and truncates the conversation to match. **Rewind modifies files on disk — reverted changes are lost unless committed to git.**”
- **Compaction:** “`/compact [context]` compresses the conversation history,” with auto-compaction at `[session] auto_compact_threshold_percent` (default 85); usage via `/context` or `/session-info`.
- **Todos:** “the agent keeps a structured todo list” with statuses pending / in progress / completed / “cancelled (when a step is dropped)”; `Ctrl+T` opens the pane, and “The list is part of the session: resume the same session and the todos return.”
- **Housekeeping:** `/sessions`, `/rename`, `grok sessions list|search|delete`, `grok export <id> [file]` (Markdown; `--clipboard`), `grok import` (“Import sessions from Claude Code”).

## Worktrees

“A worktree session runs in an isolated copy of your repository… Worktrees require a git repository, live under `~/.grok/worktrees/<repo>/<name>`, and start from your current HEAD, **including uncommitted changes**. Subagents can also request worktree isolation when the parent delegates parallel work.” Start with `grok -w`, `grok --worktree=feat "..."`, `-w --ref main` (“clean checkout of the ref”), or resume into one (`-w -r <id>`); in the TUI `/fork --worktree`, `Ctrl+W` on the welcome screen, or `Ctrl+W` in the dashboard. “A worktree is a real git checkout, detached at its base commit; land changes with ordinary git.” Housekeeping is manual: “ending or deleting a session leaves its worktree in place, and `gc` runs only when you invoke it” — `grok worktree list|show|rm|gc` (`gc --max-age 7d` expires idle ones).

## Subagents and personas

“Subagents are independent child sessions with their own context. They return a summary to the parent when finished.” Built-in types: `general-purpose` (“Default full-capability child”), `explore` (“Read, list, and search only (no shell, no edits)”), and `plan` (“Drafts an implementation plan (no shell, no edits)”). “Add or override types under `.grok/agents/` or `~/.grok/agents/`.” Manage with `/config-agents` (alias `/agents`) or `/personas`. “Personas are behavioral overlays only (tone, focus, contracts); define them under `[subagents.personas]` or `.grok/personas/*.toml` / `~/.grok/personas/*.toml`.” The settings reference shows `[subagents]` `enabled` toggle, `[subagents.toggle]` per-type switches, and `[subagents.models]` per-type routing. On defaults the pages differ: the subagents page states they are “Enabled by default when the setting is unset,” while the settings reference lists `GROK_SUBAGENTS` with default `0` (“Enable subagents / the task tool (`1`/`0`)”) — treat the TOML switch as the documented master control and set `GROK_SUBAGENTS=1` explicitly for env-driven environments. `GROK_GOAL` (default `1`) gates `/goal`; when cancelling a turn with running subagents, `[ui] cancel_subagents_on_turn_cancel` picks `ask` / `always_stop` / `always_continue`.

## Cross-session memory, workflows, and background tasks

Cross-session memory is off by default (`[memory] enabled`, `GROK_MEMORY`); when on, the shell provides `/flush` (“Flush conversation memory to disk now”), `/memory` (“Browse, view, and manage your memories”), and `/dream` (“Run memory consolidation”), plus `/remember <note>` and `grok memory clear [--workspace|--global|--all]`.

“Workflows orchestrate a bounded set of subagents in the background: fan out work, verify, then return a result. They are on by default.” `/create-workflow` authors, smoke-checks, and saves `.grok/workflows/<name>.rhai` (or `~/.grok/workflows/` for all projects); run with `/workflow <name> [args]` or `/<name>`, with `pause`/`resume`/`stop`; `/workflows` is the live run dashboard; `/deep-research <query>` starts a built-in research workflow. Disable with `[workflows] enabled = false` or `GROK_WORKFLOWS=0`.

Background tasks (`Ctrl+G` pane, `/tasks` snapshot): the agent starts “dev servers, builds, and other long-running commands as background tasks on its own”; `Ctrl+B` demotes a running foreground command; `/loop [interval] <prompt>` schedules recurring prompts — “The interval accepts `Ns` (minimum 60), `Nm`, `Nh`, and `Nd`… Loops expire after 7 days, and at most 50 scheduled tasks can be active at once.” Monitors attach to a script “each line the script prints becomes a notification in the conversation.” Queued prompts: “Prompts submitted while a turn is running are queued, not dropped” (`Ctrl+;` panel, `/queue`). This is distinct from todos: todos “track planned multi-step work rather than running processes.”

## Agent Dashboard

“The dashboard is a fullscreen overview of every session: which agents need input, which are working, and which are done,” opened with `Ctrl+\`, `/dashboard`, or `grok dashboard`. Rows group by “Needs input, Working, Idle, Inactive, Completed, Failed” (`Ctrl+G` regroups by directory); typing replies inline (idle agents get it immediately, busy ones queue it), “Permission prompts and questions can be answered inline with the number keys,” `Ctrl+L` changes the working directory for new agents, and `Ctrl+W` “toggles whether they start in a git worktree.” Search is `Ctrl+/` with `a:<name>` / `s:<state>` filters; pins/rename/stop are `Ctrl+T`/`Ctrl+R`/`Ctrl+X`. Settings persist under `[dashboard]`; disable with `enabled = false` or `GROK_AGENT_DASHBOARD=0`.

## Official sources

- [Sessions](https://docs.x.ai/build/features/sessions) · [AGENTS.md](https://docs.x.ai/build/features/project-rules) · [Subagents](https://docs.x.ai/build/features/subagents) · [Worktrees](https://docs.x.ai/build/features/worktrees)
- [Background Tasks](https://docs.x.ai/build/features/background-tasks) · [Agent Dashboard](https://docs.x.ai/build/features/dashboard) · [Modes and Commands](https://docs.x.ai/build/modes-and-commands) · [Settings Reference](https://docs.x.ai/build/settings/reference)
