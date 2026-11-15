# Approvals, sandbox, and sessions

**Verified:** 2026-11-15

## Brave mode

“Brave mode controls how much Junie CLI relies on user approval before running potentially sensitive actions. It has three levels that you can cycle through with the `/brave` slash command or the Ctrl+B shortcut”:

- **Off** — “asks for approval for every potentially sensitive action that is not on the Action Allowlist. This is the most conservative behavior.”
- **Auto** — “classifies terminal commands with a safety check and automatically approves the ones it considers safe, while still asking for approval for risky or unrecognized commands and for other sensitive actions.”
- **On** — “executes all potentially sensitive actions without user approval.”

`--brave` turns it on (interactive mode only); config.json `brave` sets the default. The IDE plugin page adds: “using brave mode is not recommended. Opt for adding actions to the Action Allowlist whenever possible.” Brave-mode preference is remembered separately for sandboxed and non-sandboxed sessions.

## Action Allowlist (CLI)

When not brave, Junie asks “before running terminal commands, MCP tools, and other types of actions that are considered to be sensitive.” The “→ Always allow” option on a prompt adds the command to `~/.junie/allowlist.json`, editable by hand. Five CLI action types: `fileEditing` (editing files outside the project, and build scripts inside or outside), `executables` (terminal commands), `mcpTools`, `readOutsideProject`, `readSecretFile`. `defaultBehavior` (“ask”) and `allowReadonlyCommands: true` are top-level knobs. Rules carry `prefix` (literal) or `pattern` (glob: `*`, `**`, `?`, `[abc]`, `[!abc]`) plus `action`: `allow` or `ask`. “Rules are evaluated top to bottom, with the first match taking precedence.”

Matching is command-aware: “A rule allows a terminal command only when every command it runs is allowed” — each part of a chain (`cd ./src && ls`) is checked separately, nested substitutions are checked “at any nesting depth” (so `git status $(curl … | sh)` still asks), and “a command that spans several lines always asks for approval” (quoted breaks and trailing `\` continuations are single commands).

## Sandbox mode (EAP; macOS and Linux only)

“Sandbox mode wraps every agent-issued terminal command in an OS-level sandbox (sandbox-runtime, srt) and restricts native edit tools.” Enable with `junie --sandbox` or toggle with `/sandbox` (ACP also accepts `/sandbox on|off`; `--sandbox` locks the toggle). The toggle belongs to the current session; “new sessions start with sandbox mode off unless you use `--sandbox`; resuming a saved session restores its sandbox setting only after successful runtime verification.” Activation “validates the runtime and policy; a failed activation leaves sandbox mode off,” and “You can always turn sandbox mode off, even if the runtime is broken.”

**What is restricted:** “terminal commands and native edit tools (including search_replace, create, apply_patch, and undo_edit) can write only to your project directory, its git directory, temp directories, and a curated set of common tool caches (~/.gradle, ~/.m2, ~/.npm, and similar). Reads are unrestricted except for ~/.ssh, which is unreadable apart from config and known_hosts.” Git-directory metadata in linked worktrees, submodules, and separate git dirs is auto-allowed after verification (“Junie asks git where its directories are and automatically allows them”), and `${GIT_DIR}`/`${GIT_COMMON_DIR}` can be deny-listed.

**Network:** “Network access is unrestricted by default: agent commands reach the network directly, while the filesystem restrictions stay fully in force.” Filtering is opt-in — `"unrestricted": false` plus `allowedDomains` (`example.com` / `*.example.com` forms; no ports, schemes, or paths), with `deniedDomains` narrowing further. “Unrestricted mode requires the JetBrains build of srt; an upstream npm runtime can be used only with the network filter enabled.”

**Policy file:** `~/.junie/sandbox.json`, created on first enable with defaults and “Junie never automatically rewrites this file.” “Only the global ~/.junie/sandbox.json is used; a project-local .junie/sandbox.json is ignored (with a warning), so a cloned repository cannot widen its own sandbox.” Supports `${PROJECT_DIR}`, `${TMPDIR}`, `${GIT_DIR}`, `${GIT_COMMON_DIR}`, `${SSH_AUTH_SOCK}`, `~` in path entries — “Anything else must be a literal absolute path — glob patterns are not supported” — plus `allowWrite`/`denyWrite`/`denyRead`/`allowRead`, `network.allowUnixSockets` (per-path, macOS), `network.allowAllUnixSockets` (Linux), `network.allowSystemCertificateValidation`, and `network.unrestricted`. The policy file sits on the sandbox’s own deny-write list: the agent may edit it only via a native edit tool that explicitly names the file, “Junie CLI shows the exact path and before/after content and requires you to give a fresh Allow once approval for every change, including undo. Your request alone is not approval; brave mode and saved approvals cannot bypass this confirmation.”

**SSH and Go tools:** with an ssh-agent running (including 1Password/ssh-add), “git can authenticate through the agent without reading a private key”; without an agent “pushing over SSH fails by design. Use an https remote or the gh CLI.” Go binaries (gh, kubectl, gcloud, terraform) validate certificates through the system trust daemon, so the default policy sets `network.allowSystemCertificateValidation: true` “so gh works out of the box” — “The trade-off is that the trust daemon itself becomes a channel a malicious command could push data through.”

**srt’s own protections (not allowlistable):** “notably .git/hooks, .git/config, shell rc files, .mcp.json, and .vscode / .idea directories. Commands like `git remote set-url` therefore fail while the sandbox is on.” On macOS, Unix sockets (e.g. `/var/run/docker.sock`) are blocked unless listed, and “Apple Events are denied, so open and osascript do not work inside the sandbox.”

**Scope limits:** “MCP servers, hooks, and operations performed directly by Junie CLI outside those tools are not isolated by this sandbox. Reachable host services and writable caches can also act outside its restrictions. It is not hardened isolation like a container or VM.” Linux needs system bubblewrap, socat, and ripgrep; Ubuntu AppArmor setups offer a reviewable `junie-sandbox-setup.sh` that grants `/usr/bin/bwrap` user-namespace permission — “Junie still runs as your normal user. It does not disable AppArmor globally.” Violation monitoring surfaces recorded denials (macOS sandbox log; Linux `/usr/bin/strace` without admin) — “This is diagnostic monitoring, not a complete audit… An empty report does not prove that nothing was blocked.”

## Parallel sessions and worktrees

“Junie can keep multiple live sessions in one interactive terminal.” `/new [prompt]` starts another session while the current one “stays live in the background”; `/history` opens Task history (live + saved sessions, Current project / All sessions tabs, Ctrl+X delete). “Switching to a live session does not restart it.” Cross-instance live sessions show dimmed and cannot be opened from the wrong instance (“This prevents two terminal UIs from controlling the same live session”). “Switching away from a session — via /new, a /history selection, or any other session switch — does not end the outgoing session: it continues running in the background and its SessionEnd hook is not fired on the switch.”

“Parallel sessions share the file system of the project or worktree they are running in. When you want several sessions to make code changes at the same time, use Git worktrees.” `/worktree` opens a menu to switch or create — Junie names them `<project>-junie-wt-01`, `-02`, … “as a sibling directory of your project” (“Make sure the parent directory is writable”), and “resets the current session state for the new worktree.” With uncommitted changes present, Junie asks to “Transfer changes to workspace” (via `git stash`) or “Start with a clean workspace”; a conflicted stash is “left… so you can resolve them manually.” “Worktree support requires a git repository.”

## Plan mode

“Junie CLI analyzes the codebase with read-only operations and produces a design document for the task before any code is written.” Toggle with Shift+Tab (cycles default → Plan → Debug) or `/plan [prompt]`; start directly with `--plan` (combine with `--prompt` to auto-submit). “Plan mode applies to the next prompt you submit. After each submitted prompt, Junie CLI returns to the default mode.” The proposed plan pauses for review — Confirm and implement / View the entire plan (Ctrl+P plan view with tabs like Requirements, Technical design, Testing, Delivery steps) / Open `<plan-file>` / “Save the plan and stop.” “The plan is treated as a living design document: when requirements change in the same session, the plan is updated alongside them.” `/plan close` (ACP) exits planning “without deleting the plan file.”

## Debug mode

“Junie CLI acts as an AI debugging assistant that manages breakpoints, inspects runtime state, and evaluates expressions against a live debugger session in a connected JetBrains IDE.” `/debug` or Shift+Tab cycle; session control (launch/attach/resume/step), line and exception breakpoints, variable and call-stack inspection, thread/frame switching, expression evaluation. “Junie CLI does not modify source code while in Debug mode unless you explicitly ask.” Requires an IDE with debugging support — without one: “Debug mode requires an IDE with debugging support connected.” Like plan mode, it applies to the next prompt and then returns to default.

## Local code review

`/review [target]` runs “a tailored subagent with optimized resource consumption for code review tasks: instead of running a full agent session, it uses a more focused system prompt and a subset of read-only tools, focusing on the changed lines of code.” The agent “never edits files, runs builds or tests, creates files, or commits and pushes changes,” but “follows your guidelines and skills,” and the review is “its own session that can be resumed.” Without an argument, a wizard offers: Changes compared to main, Last Commit, Staged Changes, Unstaged Changes, plus a free-text field (commits, branches, ranges like `origin/main..HEAD`, “the last 2 commits”, instructions like “focus on error handling” — “Junie CLI asks a lightweight model to work out the target from your text”). Findings land in a review screen grouped by file, accept/dismiss per item; “Critical findings… are prefixed with a [CRITICAL] label.” “/review requires the current project to be a Git repository.” Headless: `junie --review` (optionally with instructions).

## Session management extras

- **Transcripts:** Ctrl+O opens “a continuously updated transcript.md file stored next to the session’s events.jsonl file”; subagent transcripts live in the session’s `subagents` folder. `/settings` can switch to the built-in Transcript view.
- **Branching:** `/branch [name]` copies the current conversation into a new session and switches to it; `--session-id <id|junie://sessions/<id>>` resumes, `--fork-session` copies and runs without touching the original.
- **Teleport:** `/teleport [path]` packs “the conversation and terminal output” into a zip (`~/.junie/teleport/` by default); `/continue [archive]` restores it — “The project must be a clean checkout of the same repository when the archive carries code.”
- **Links:** `/link` prints `junie://sessions/<id>`; `junie junie://sessions/<id>` resumes it. “Interactive TUI only; the link references a local session in the same Junie Home. No data is uploaded.”
- **Exit:** `/quit` exits “while staying logged in”; Ctrl+C twice quits; `/restart` reopens the session (managed launcher only; direct binary/JAR runs error). Titles via `/title` persist and are “not overwritten by the agent’s automatic name.”

## Official sources

- [Action Allowlist (CLI)](https://junie.jetbrains.com/docs/action-allowlist-junie-cli.html) · [Plan mode](https://junie.jetbrains.com/docs/junie-cli-plan-mode.html) · [Debug mode](https://junie.jetbrains.com/docs/junie-cli-debug-mode.html) · [Parallel sessions and worktrees](https://junie.jetbrains.com/docs/junie-cli-worktrees.html) · [Code review agent](https://junie.jetbrains.com/docs/junie-review-agent.html) · [Slash commands reference](https://junie.jetbrains.com/docs/slash-commands.html) · [Quickstart (sandbox section)](https://junie.jetbrains.com/docs/junie-cli.html)
