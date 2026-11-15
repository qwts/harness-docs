# Surfaces: CLI, VS Code, JetBrains, and Desktop

**Verified:** 2026-11-15

## CLI

Install with `npm install -g @kilocode/cli` (Kilo CLI 1.0), producing the `kilo` command. The docs state the lineage plainly: “The Kilo CLI is a fork of OpenCode and supports the same configuration options,” and the installed CLI “is built from Kilo-Org/kilocode.” The legacy `kilo console` browser interface “is deprecated and will be removed in a future release.”

Core invocations:

- `kilo` — interactive TUI (“Interactive mode is the default mode when running Kilo Code without the `--auto` flag”).
- `kilo run [message..]` — non-interactive runs. Approval behavior is explicit: “Without `--auto`, a non-interactive run cannot prompt for approval and auto-rejects any permission request it receives. If a run auto-rejected at least one request, it exits 1 with a stderr diagnostic naming the cause, since the task likely did not complete. Pass `--auto` for autonomous use.” The flag itself is described as “auto-approve permissions that are not explicitly denied (dangerous!)”.
- `kilo run` flags: `-m/--model`, `--agent`, `-c/--continue`, `-s/--session`, `--fork`, `--cloud-fork`, `--format` (`default`|`json`), `-f/--file`, `--title`, `--variant`, `--thinking`, `--dir`, and attach options (`--attach URL` with `-p/--password`/`-u/--username`).
- Organization selection: “There is no `--org` or `--team` flag on `kilo run`. Instead, the organization is determined from” `KILO_ORG_ID` first, then “Persisted selection from the last `/teams` pick” stored in the CLI auth file.
- Subcommands include `serve`, `acp`, `mcp add|list|auth|logout|debug`, `session list --search|--all`, `db query` (SQL over session storage), `config`, `plugin`, `github`, `pr`, `cloud`, `remote`, `daemon`, `models`, `roll-call`, `auto`, `stats`, `export`/`import`, `auth`, `agent`, `upgrade`, `uninstall`, `profile`.

Slash commands cover sessions (`/sessions`, `/new`, `/fork`, `/timeline`, `/copy-session`, `/move`), history (`/undo`, `/redo`, `/diff`), imports (`/resume-claude`, `/resume-codex` — reading `~/.claude/projects/` and `~/.codex/sessions/`), model/agent management (`/models`, `/agents`, `/mcps`), gateway (`/profile`, `/teams`, `/remote`), and built-ins (`/init` — create/update AGENTS.md, `/review` — code changes, commits, or PR URLs, `/goal [objective|pause|resume|clear]`). `/reload` “reloads every instance of the project, including the main checkout and sibling worktrees. Kilo refuses the reload while any session in the project is running.” `/auto-approve` (aliases `/approve-all` etc.) toggles auto-approve “for all permission prompts (saved to global config)”; `/privacy` “blurs PII in the TUI.”

TUI alerts are configured in `~/.config/kilo/tui.jsonc` (or project `.kilo/tui.json`) under an `attention` key — `enabled`, `notifications`, `sound`, `volume` (0–1), and per-event `sounds` overrides (`question`, `permission`, `error`, `done`, `subagent_done`). “CLI attention alerts are disabled by default.” `KILO_DISABLE_MOUSE=1` disables TUI mouse controls.

## VS Code extension

The extension ships in the Marketplace pre-release channel (`kilocode.kilo-code`) and “includes its own embedded runtime. No separate Kilo CLI installation is required” — it “starts its embedded runtime from the extension and applies the relevant VS Code network settings to that runtime.” The rebuilt sidebar uses a “SolidJS-based UI.”

Editor-integrated features:

- **Autocomplete** — FIM completions via the gateway (Codestral); Tab to accept, with a manual Cmd+L/Ctrl+L trigger gated behind the VS Code setting `kilo-code.new.autocomplete.enableSmartInlineTaskKeybinding` (“disabled by default”). Status-bar item shows state (active/snoozed) and cumulative cost; snooze/unsnooze by clicking. “The extension automatically detects if GitHub Copilot inline suggestions are enabled and warns you about potential conflicts.”
- **Code Actions** — Add to Context (Cmd+K Cmd+A), Explain Code, Fix Code, Improve Code, plus terminal context-menu actions (Add Terminal Content, Fix Command, Explain Command). “Code Actions are a VS Code extension feature and are not available in the CLI/TUI”; they route to the active Agent Manager session when one exists.
- **Git commit generation** — “This feature only analyzes staged changes”; conventional-commits format by default, customizable via `commit_message.prompt` in project config (Settings → Commit Message). Also extension-only.
- **Browser automation** — Playwright-driven; settings under Settings → Web Tools → Browser Automation (headless default disabled, system Chrome default enabled, with Playwright Chromium fallback). Requires a capable model. The experimental Integrated Browser under Settings → Experimental “is a different feature” showing local previews inside Agent Manager.
- **Task todo lists** — managed by the `todowrite` tool (“Each update replaces the entire checklist”); surfaced as a Session Dock progress chip, an interactive tool block (editable before approval), and a background “REMINDERS” table. Statuses: pending, in progress, completed.
- **Message feedback** — thumbs up/down “only available when telemetry is on”; sends the rating, provider/model ID, and gateway request ID, nothing else.
- **Checkpoints** — automatic snapshots using shadow-git (see below).

Background-agent strip states: Running, Done, Cancelled, Error, Needs input, with Stop all (N) and Continue in background; queued follow-up messages wait while a session runs. Deep links: `vscode://kilocode.kilo-code/kilocode/switch?model=<modelID>` and `…?agent=<agentName>`.

## Checkpoints (shadow git)

Checkpoints snapshot the workspace after changes using a shadow git repository — snapshots are stored “as raw tree hashes (not refs or commits),” so “older snapshots may be pruned by garbage collection even if a session still references them”; “A background process runs `git gc --prune=7.days` every hour.” Revert All restores the pre-change state and deletes files the agent created; Redo All restores “using `git checkout-index`.” “When using the Agent Manager with git worktrees, each worktree gets its own isolated snapshot repository,” preventing cross-worktree interference while sharing underlying Git objects for storage efficiency. “If Git is unavailable or the workspace is not a Git repository, Kilo skips checkpoints automatically.”

## JetBrains plugin

“Kilo Code v7 for JetBrains is officially available. It uses a native JetBrains interface, works well in remote development split mode, and does not require Node.js.” Supported IDEs (9): IntelliJ IDEA, WebStorm, PyCharm, PhpStorm, GoLand, Rider, CLion, RubyMine, DataGrip. Plugin updates come from the JetBrains Marketplace; the v7 EAP repository URL (`https://kilo-org.github.io/kilocode/jetbrains/updatePlugins.xml`) should be removed from “Settings → Plugins → Manage Plugin Repositories” to avoid staying on EAP updates. “Open Settings → Tools → Kilo Code to configure the plugin. Shared agent settings use the same kilo.jsonc files as the CLI and VS Code extension; IDE-specific options such as GitHub integration and worktree indexing stay in the plugin.” The JetBrains plugin’s one permission deviation: `edit` prompts by default (“a plugin default, not an environment override”).

## Desktop

Kilo Desktop wraps the same agent with workspace-scoped feature panels: Notebooks (interactive Python), in-app browser, Terminal, Git integration, Workspace board (sessions as columns), Local inference, Conda environments, and a Files browser.

## Official sources

- [CLI](https://kilo.ai/docs/code-with-ai/platforms/cli) · [CLI reference](https://kilo.ai/docs/code-with-ai/platforms/cli-reference) · [VS Code](https://kilo.ai/docs/code-with-ai/platforms/vscode) · [JetBrains](https://kilo.ai/docs/code-with-ai/platforms/jetbrains) · [Desktop](https://kilo.ai/docs/desktop/overview) · [Shell integration](https://kilo.ai/docs/automate/extending/shell-integration) · [Checkpoints](https://kilo.ai/docs/code-with-ai/features/checkpoints)
