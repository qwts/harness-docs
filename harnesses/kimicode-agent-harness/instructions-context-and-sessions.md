# Instructions, context, and sessions

**Verified:** 2026-11-15

## AGENTS.md

Kimi Code follows the AGENTS.md convention. Locations: the project's `AGENTS.md` or `.kimi-code/AGENTS.md` (project instructions), the global `$KIMI_CODE_HOME/AGENTS.md` (Kimi-specific instructions applied to every session), and `~/.agents/AGENTS.md` (“generic `.agents` resources stay under the real OS home so they can be shared across tools”). `AGENTS.md` is among the files the `[watch]` filesystem watchers hot-reload when enabled (watching is off by default per the whats-new log — see the configuration page). The VS Code extension's `/init` command will “scan the project and generate documentation.”

## SYSTEM.md and agent files

The system prompt is overridable: “$KIMI_CODE_HOME/SYSTEM.md permanently overrides the default main agent's system prompt; it is not part of agent-file discovery.” Template variables available in SYSTEM.md and agent prompts include `${skills}`, `${agents_md}`, `${cwd}`, `${cwd_listing}`, `${os}`, `${shell}`, `${now}`, `${additional_dirs_info}`, `${base_prompt}`, `${plugin_sections}`, and pre-composed block variables. The agents page carries an explicit trust note that project-level files “come from the repository itself, including repositories you have just cloned and do not trust yet.”

## Custom agents

Agent files are Markdown with YAML frontmatter (`name`, `description` required; `whenToUse`, `override`, `tools`, `disallowedTools`, `subagents`). Locations in precedence order — “Explicit (--agent-file) > Project > Extra > User > Plugin > Built-in”: project `.kimi-code/agents/` and `.agents/agents/`, user `$KIMI_CODE_HOME/agents/` and `~/.agents/agents/`, plugin-provided, and three built-ins. “A project-scoped file can take over a built-in agent entirely,” which requires `override: true`. The CLI's `--agent` selects a named agent while `--agent-file` loads a single file at the highest priority; the two are mutually exclusive.

## Skills

Skills are `SKILL.md` files discovered from project `.kimi-code/skills/` and `.agents/skills/`, user `$KIMI_CODE_HOME/skills/` and `~/.agents/skills/`, `extra_skill_dirs`, and built-in product skills. Directory form requires frontmatter — “both name and description must be explicitly provided. Omitting either one will cause parsing to fail” — while the flat single-file form falls back to the filename for `name` and the first body line (≤240 chars) for `description`. Types are `prompt` (default), `inline`, and `flow`; other fields: `whenToUse`, `disableModelInvocation`, `arguments`. `$ARGUMENTS`, `$0`/`$1`, and `${KIMI_SKILL_DIR}` are substituted; “If the body contains no argument placeholders, text passed at invocation is appended to the end of the body” as `ARGUMENTS: <text>`. Invocation is `/skill:<name>`, the `/name` shorthand, dotted sub-skills, nesting up to three levels, and `merge_all_available_skills` to combine everything instead of shadowing by precedence. `--skills-dir` “Replaces the automatically discovered user and project directories for this launch only.”

## Sessions, undo, fork, and history

Session state lives in `$KIMI_CODE_HOME/sessions/<workDirKey>/<sessionId>/`: `state.json` for resumable state, `agents/main/wire.jsonl` recording the wire traffic including request traces, `agents/main/plans/` for plan files, `agents/agent-N/` for subagents, `tasks/` for background tasks, and `cron/` for scheduled jobs. `session_index.jsonl` indexes sessions and `user-history/<md5(workDir)>.jsonl` stores typed input history. Resume is `-c/--continue` (most recent) or `-S/--session [id]`; a hidden `-r/--resume` selector exists. Mode flags interact with resume narrowly: “In non-prompt mode, --yolo and --plan cannot be combined with --continue or --session,” and for agents “The agent is bound at session creation, and resuming restores the bound agent automatically, so no flag is needed (or allowed) on resume.” `/undo [count]` reverts turns, `/fork` branches a session, `/title` renames (≤200 chars), `/compact [hint]` compresses context, and `/export-md` writes Markdown. `kimi export` ships a session archive (`-o` path, `-y`, `--no-include-global-log`).

`kimi import-sessions` migrates history from Claude Code and Codex: “Choose the last 15 days (the default) or the last 30 days. Each run imports at most the newest 100 sessions,” honoring `CLAUDE_CONFIG_DIR` and `CODEX_HOME`. The migration FAQ notes “OAuth login credentials and MCP service authorizations are not copied, so you will need to run /login again and re-authorize MCP servers after migrating.” `/import-from-cc-codex` is the in-CLI skill equivalent.

## Data lifecycle details

`credentials/` is written atomically — “dir 0700, files 0600” — via temp-file, fsync, rename. `bin/rg` and `bin/fd` are managed binaries downloaded by the CLI. `logs/` keeps rotating logs, `updates/` staged auto-updates. Goal queues live in `upcoming-goals.json` (`/goal next`), invisible to the agent until promoted.

## Official sources

- [Agents](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/agents.html) · [Skills](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/skills.html) · [Slash commands](https://www.kimi.com/code/docs/en/kimi-code-cli/usage/slash-commands.html) · [Sessions](https://www.kimi.com/code/docs/en/kimi-code-cli/usage/sessions.html) · [Data locations](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/data-locations.html) · [Migration](https://www.kimi.com/code/docs/en/kimi-code-cli/migration.html)
