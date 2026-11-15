# Command Code corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| Configuration lives in `~/.config/command-code/`. | ❌ FALSE | The config root is `~/.commandcode/` (JSON files); project files are `.commandcode/` in the repo plus `.mcp.json` at the root. |
| Settings files are YAML or TOML. | ❌ FALSE | All configuration files are JSON. |
| A project allow rule overrides a user-global deny. | ❌ FALSE | Rule lists “union across every scope rather than being overwritten,” and “deny wins, then ask, then allow”; “specificity doesn’t reorder anything - a broad deny still blocks a narrower allow.” |
| `--permission-mode yolo` is enough for headless runs that write files. | ❌ FALSE | Headless mode “blocks file writes, file edits, and shell commands unless you pass `--yolo` or `--dangerously-skip-permissions`”; the mode flag alone “does not lift that block.” |
| Yolo mode is suitable for production work. | ❌ FALSE | The docs describe it as “Permission bypass: do basically everything without asking” and “throwaway environments only”; `permissions.disableBypass` can make it “unenterable,” neutralizing the CLI flags at entry. |
| `/mode yolo` switches into yolo. | ❌ FALSE | “Yolo is deliberately not slash-switchable - slash commands are agent-invokable”; use shift+tab or launch with `--yolo`. |
| Approvals are always persisted to `settings.json`. | ❌ FALSE | Interactive approvals write to `settings.json` normally, “and to `settings.local.json` when started with `--local`”; safety-forced prompts “offer only one-time Yes/No” and “are never cached.” |
| Command Code supports many hook event types across the lifecycle. | ❌ FALSE | “Only four events are recognized: PreToolUse, PostToolUse, Stop, and SessionStart”; the docs point broader lifecycle needs at Mods. |
| A PostToolUse hook can undo a tool call. | ❌ FALSE | PostToolUse `decision: "block"` is “Advisory retry signal to the model. The tool already ran, so nothing is un-done.” |
| Stop hooks can retry a turn indefinitely. | ❌ FALSE | Stop-hook blocking “forces a revision, capped at 3 retries” per turn, with `stop_hook_active` loop protection. |
| Hooks run in plan mode. | ❌ FALSE | “Hooks are skipped entirely in plan mode” — the mode is read-only. |
| taste-1 is an LLM. | ❌ FALSE | The docs state “taste-1 is not an LLM. It does not generate text and it does not answer prompts”; it classifies signals and scores confidence, while LLMs handle the input-processing and learning-writing steps. |
| Taste uploads your code to the cloud for training. | ❌ FALSE | Taste is local markdown at `.commandcode/taste/`; “Taste learning runs locally and stores preferences as structured rules - not code snippets”; remote packages require an explicit `npx taste push` (public only with `--public`). |
| Taste learning costs extra per learning. | ❌ FALSE | taste-1 is “Included in every plan at no extra charge”; only the LLM steps (input processing, writing learnings) bill normally. |
| Command Code reads `CLAUDE.md`. | ❌ FALSE | “Command Code reads AGENTS.md, not CLAUDE.md”; `/import` migrates other agents’ files and rewrites references. |
| Memory tiers replace each other; only the nearest file loads. | ❌ FALSE | “Every one that exists is loaded; they add up, they don’t replace each other” — user, then project (first of AGENTS.md/.commandcode/AGENTS.md), then subdirectory files on demand. |
| Compaction rewrites the session transcript. | ❌ FALSE | Sessions are “a single append-only JSONL file” whose entries form a tree; compaction appends an entry — “The on-disk log is never rewritten.” |
| Checkpoints are kept forever. | ❌ FALSE | “Checkpoints are kept for 30 days, capped at 200 backups per session,” with file backups “capped at 10MB per file.” |
| `--continue` can resume sessions from other directories. | ❌ FALSE | The catalog is project-scoped: “`/resume` and `--continue` only ever see sessions from the directory you launched Command Code in.” |
| Subagents can spawn their own subagents. | ❌ FALSE | “Subagents can’t start their own subagents - the agent tool is removed from their tool set.” |
| Agents can be named `explore`, `plan`, `review`, or `general`. | ❌ FALSE | Those are reserved names; “files using them are ignored.” |
| Subagents run unlimited turns. | ❌ FALSE | `maxTurns` defaults to `subagentMaxTurns` (else 200), range 1–1000; headless `--max-turns` defaults to 100 and “exits with code 8 if the cap is hit.” |
| Skills always execute their inline shell commands. | ❌ FALSE | `disableSkillShellExecution: true` stops it — “so opening a skill never shells out” — replacing every placeholder with “[shell command execution disabled by policy].” |
| Command Code discovers `.claude/skills/` directories. | ❌ FALSE | The documented roots are `.commandcode/skills/`, `.agents/skills/`, `~/.commandcode/skills/`, and `~/.agents/skills/` (plus settings paths); no `.claude/` root is documented. |
| Project MCP servers override local ones. | ❌ FALSE | “When the same server name exists in multiple scopes, local overrides project, which overrides user.” |
| Mods run in a sandbox. | ❌ FALSE | “There is no sandbox. A mod is code that runs with your permissions” — the ModApi is also marked Experimental. |
| Mods can silently answer the permission prompt. | ✅ TRUE | `onPermissionRequest` “answers a permission prompt before the person sees it” and returns `allow`, `deny`, or undefined — it “fires only where the engine would ask a human,” deny wins across hooks, and on safety-forced prompts “‘deny’ works but ‘allow’ is ignored.” What mods can’t do is redraw the dialog: `cmd.ui.override` covers the spinner, tool rows, user messages, and command output, so “the permission prompt (and every other approval dialog) is not on the list and never will be.” |
| The desktop app has CLI feature parity. | ❌ FALSE | Desktop lacks headless mode, `/share`, scheduled tasks, start-up flags, subcommands, the `dont-ask` mode, and several slash commands; `/config`, `/theme`, and `/mcp` map to Settings pages. |
| Desktop commits include only the agent’s changes. | ❌ FALSE | “A commit includes all current project changes, not only changes made by the agent.” |
| Telemetry cannot be disabled. | ❌ FALSE | `DO_NOT_TRACK=1` stops exporters, and local-only mode guarantees “No telemetry, anywhere”; the security page states no tracking “without your knowledge.” |
| BYOK providers can be configured per project or repo. | ❌ FALSE | “BYOK providers are set only on the user-level, with config stored globally at ~/.commandcode/providers.json.” |
| Command Code is fully Claude Code-compatible. | ⚠️ PARTIAL | Permission-mode aliases (`manual`, `acceptEdits`, `bypassPermissions`, `standard`), `CLAUDE_*` skill placeholders, and `.agents/skills/` roots are honored — but the config root, memory file (AGENTS.md only), and skill priority differ, and `.claude/` locations are not documented. |
| `/share` publishes the thread publicly. | ⚠️ PARTIAL | Link sharing “is being rebuilt”; the working path is `/share gist` (formats html, jsonl, md), which posts a secret GitHub gist via the `gh` CLI. |
| Local-only mode confines file access to the project directory. | ⚠️ PARTIAL | The security page’s “Local file access” section says Command Code reads only the project directory, `--add-dir`/`/add-dir` additions, and `~/.commandcode/` (“It does not read files outside these boundaries”), but the permissions reference makes the boundary permission-dependent: outside directories “first ask to admit the directory into the session’s root set” and “yolo grants the directory silently,” the OS temp dir “is granted silently in every mode,” registered skill directories “are readable without a grant,” and “absolute paths inside a shell command string are not external-dir gated.” Local-only mode blocks backend/network traffic, not the filesystem. |
| Yolo is the recommended way to work. | 💬 OPINION | The docs position it as “throwaway environments only”; mode choice is workload-dependent, with `dont-ask` documented as “the CI/CD mode.” |
