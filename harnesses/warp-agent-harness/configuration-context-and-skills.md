# Configuration, project context, rules, and skills

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main)

## The CLI settings file

“The Warp Agent CLI keeps its configuration in a plain TOML settings file on your machine”:

- macOS: `~/.warp_cli/settings.toml`
- Linux: `~/.config/warp-terminal/cli/settings.toml` (respects `$XDG_CONFIG_HOME`)
- Windows: `%LOCALAPPDATA%\warp\Warp\config\cli\settings.toml`

“CLI settings are local to your machine and are never synced to the cloud. They are also independent from the Warp app's settings: the app and the CLI keep separate settings files, so changing one never affects the other.” The file is watched at runtime: “The CLI watches the file while it's running and reloads most values as you save” (theme applies on next `/theme` or restart). “If a value is invalid, the CLI logs the problem and uses the default for that setting instead. If the file can't be parsed at all, the CLI starts with default settings.”

Edit it three ways: directly, via slash-command panels (`/theme`, `/statusline`), or by asking the agent — “The CLI ships with a bundled skill and a schema of every available setting, which the agent uses to find the right key, validate the value, and update the settings file for you.”

Surface keys include `[appearance]` (`theme` — “`/theme auto` … Matches the host terminal's background (default)”), the statusline (“the auto-approve indicator, the active model, the working directory, the Git branch, and the Git diff status inside a repository” by default, with optional conversation-usage, context-window, task-list, PR, date/time, and voice chips), `appearance.zero_state` start-screen options (“a rotating ASCII object alongside sections for your account, the changelog, project info, and MCP servers”), `general.autoupdate_enabled`, and permission fields under `[agents.execution_profiles.*]` (see [permissions and approvals](permissions-and-approvals.md)).

## Rules

“Warp's **Rules** feature lets you create reusable guidelines that inform how agents respond to your prompts.” Two types:

- **Global Rules** — “A rule file at `~/.agents/AGENTS.md` applies across all projects on your machine.”
- **Project Rules** — “stored in an `AGENTS.md` file (or `WARP.md` for backwards compatibility),” placed “in the root of your repository” and “in subdirectories for more targeted guidance.”

Behavior details:

- “Warp uses `AGENTS.md` as the default project rules file. Existing `WARP.md` files are still fully supported,” and “If both `WARP.md` and `AGENTS.md` exist in the same directory, `WARP.md` takes priority.”
- “The filename must be in **all caps** for Warp to recognize it (e.g., `AGENTS.md`, not `agents.md` or `Agents.md`).”
- Application: “Warp automatically applies the `AGENTS.md` (or `WARP.md`) in the root and in the current directory,” plus a “best-effort attempt” for the rules of any subdirectory you edit files in.
- Precedence: “1. Rules in the current subdirectory's project rules file 2. Rules in the root directory's project rules file 3. Global Rules.”
- In the CLI, “When you `cd`, project rules and skills re-scope to the new directory automatically.”

## Skills

“Skills are reusable instructions for tasks such as deployment, code review, or documentation checks.” Format: “Write each skill in a `SKILL.md` file with YAML frontmatter” — `name` and `description` — with supporting files alongside. “At the start of a conversation, the agent receives the names and descriptions of available skills. It loads a skill's full instructions when the task calls for them.”

- **Locations** — project skills in repository skill directories (e.g. `.agents/skills/`), personal skills in `~/.agents/skills/`, “scoped to your current working directory.” “Project discovery follows your working directory up to the Git repository root; skills from another project are not in scope.”
- **Invocation** — natural language, or `/skill-name` directly (“so you can add extra instructions before running it”); `/skills` browses everything in scope.
- **Arguments** — “`$ARGUMENTS` - Full raw text after the skill name”; “`$ARGUMENTS[N]` - The Nth whitespace-separated argument, indexed from zero”; “`$N` - Shorthand for `$ARGUMENTS[N]`”; unmatched index placeholders are “left as-is in the skill content.” Without placeholders, extra text arrives “as a separate user message alongside the skill.”
- **Conflicts** — “Background resolution - Home-directory skills take priority, then project skills closer to the repository root.”
- **Bundled CLI skills** — `/modify-settings` (settings-schema editing) and `/tui-migrate-setup` (app-to-CLI migration). The app adds `/agent-add-mcp` for MCP server setup (see [MCP and integrations](mcp-and-integrations.md)).

## Codebase Context

“Warp indexes your Git-tracked codebase to help Agents understand your code and generate accurate, context-aware responses. No code is stored on Warp servers.” Indexing starts when a Git repo is opened; status lives in **Settings** > **Code** > **Indexing and projects**. The app starts agent conversations with `⌘+Enter` (macOS) or `Ctrl+Shift+Enter` (Windows/Linux).

## Agent questions

When the agent needs a decision, it pauses with an **Agent questions** card: “Multiple-choice questions,” single- or multi-select, an optional free-form **Other** answer, multiple questions per card, and skip support — “If you skip a question, Warp sends the skipped state to the Agent so it can continue with the context it already has.” The **Ask questions** permission controls this: “`always_ask`, `ask_except_in_auto_approve`, or `never`.”

## Notifications

“Warp delivers notifications from any supported coding agent so you always know when an agent finishes a task, encounters an error, or needs your input.” Types: **Complete** (“the agent finished its task successfully”), **Request** (“the agent is blocked and needs your input. This includes command approval, permission requests, and idle prompts”), and **Error** (“the agent encountered an error that requires your attention”). Delivered as in-app toasts (“Up to two toasts are visible at a time”), desktop alerts, and more.

## No hooks or plugins

The docs document no hook/plugin lifecycle for the Warp Agent — context enters through rules, skills, Codebase Context, MCP servers, and Warp Drive objects (prompts are the object type available in the CLI: “your saved Warp Drive prompts are available after you log in (other Warp Drive object types aren't available in the CLI)”). Cross-agent persistence is Agent Memory, which is “in **research preview** and … enabled per team for design partners.”

## Official sources

- [Customizing the CLI](https://docs.warp.dev/agents/cli/configuration) · [Rules](https://docs.warp.dev/agents/capabilities/rules) · [Skills](https://docs.warp.dev/agents/capabilities/skills) · [Codebase Context](https://docs.warp.dev/agents/capabilities/codebase-context) · [Agent questions](https://docs.warp.dev/agents/local-agents/interacting-with-agents/agent-questions) · [Agent notifications](https://docs.warp.dev/agents/capabilities/agent-notifications) · [Agent Memory](https://docs.warp.dev/agents/agent-memory)
