# Permissions, hooks, and mods

**Verified:** 2026-11-15

## Five permission modes

“A mode sets the baseline. Cycle with shift+tab, or switch with `/mode`.”

| Mode | Baseline |
|---|---|
| `default` | “Prompt before anything that changes things. Reads are free.” |
| `accept-edits` | “Do normal edits (and safe file commands) without asking.” |
| `plan` | “Read-only. Explore and plan; only the plan file may be written.” |
| `yolo` | “Permission bypass: do basically everything without asking” — “throwaway environments only.” |
| `dont-ask` | “Never ask. Run what’s pre-approved; deny the rest. The CI/CD mode.” |

shift+tab (or alt+m) cycles `default ↲ accept-edits → plan → yolo → default`; `dont-ask` sits outside the cycle and is set via settings or `--permission-mode`. Yolo is deliberately not slash-switchable — “slash commands are agent-invokable” — so `/mode yolo` points you at shift+tab or `--yolo`. `permissions.disableBypass: "disable"` (or `true`) “make[s] yolo unenterable,” enforced at every layer: the engine evaluates a requested yolo as default and the CLI neutralizes `--dangerously-skip-permissions`/`--yolo` at entry.

## Rules: deny, then ask, then allow

`permissions  holds `allow`, `ask`, and `deny` string arrays checked in that order: “deny wins, then ask, then allow.” Rules look like `Tool` or `Tool(specifier)` — `Shell(git status:*)`, `Edit(src/**)`, `Read(secrets/**)`, `WebFetch(domain:example.com)`, `mcp__github__*`. The lists **union across every scope** (“a deny in your user-global settings can’t be erased by a project file”), specificity never reorders the ladder, and matching is asymmetric: “Deny and ask match aggressively … Allow matches conservatively.” Shell parsing understands operators — `Shell(git status)` does not grant `git status && rm -rf`.

Notable edges: a removal of the filesystem root or home dir “always stops for confirmation” — the one safety prompt that survives yolo (denied in dont-ask/plan) — and three tools skip the lists entirely (`ask_user_question`, `agent`, `run_command`). Sub-agents run the same pipeline with prompts auto-allowed, while hard guards “fail closed.” Interactive approvals are remembered: file ops enable session accept-edits, shell commands persist a rule to project settings, and safety-forced prompts “offer only one-time Yes/No” so destructive approvals “are never cached.” Approvals write to `.commandcode/settings.json  normally, or `settings.local.json` when started with `--local`.

## Headless is a special case

In print mode (`cmd -p`), “It blocks file writes, file edits, and shell commands unless you pass `--yolo` or `--dangerously-skip-permissions`.” `--permission-mode yolo` sets the mode but “does not lift that block.”

## Hooks: four events

Hooks are “shell scripts that Command Code runs automatically before and after each tool call,” configured under the `hooks` key (project scope beats user scope). Exactly four events are recognized:

| Event | Fires | Can block? |
|---|---|---|
| `PreToolUse` | Before a tool runs | Yes — deny via `permissionDecision: "deny"` or exit 2 |
| `PostToolUse` | After the tool returns | No — advisory `decision: "block"` is “a retry signal to the model. The tool already ran, so nothing is un-done.” |
| `Stop` | When the assistant finishes a turn | Yes — forces a revision, “capped at 3 retries” per turn |
| `SessionStart` | Startup, resume, clear | No — injects `additionalContext`/`systemMessage` only |

Handlers are `{"type": "command", "command": "…"}` with an optional `matcher` (pipe-separated, tested against `tool_display_name`; meaningful only for the tool events) and `timeout` — “Defaults to 30, maximum 600” seconds; the settings reference additionally lists `async` and `failClosed` fields. Hooks receive tool input as JSON on stdin, run PreToolUse sequentially (a denial skips the rest) and the other events in parallel, get `COMMANDCODE_PROJECT_DIR`/`SESSION_ID`/`HOOK_EVENT`/`CWD` injected, and “are skipped entirely in plan mode.” `continue: false` with `stopReason` halts the session after the current tool batch.

## Mods: the extension layer

Mods are “TypeScript packages that let you modify almost any behavior of Command Code,” loaded from `~/.commandcode/mods/`, `<project>/.commandcode/mods/` (trust-gated), `mods.paths`/`mods.sources`, or `--mod <path>`, compiled onto the agent loop at startup via jiti. The ModApi is Experimental. The core rule: “hooks mutate, on observes” — `cmd.hooks({beforeToolCall, transformInput, beforeModelRequest, appendSystemPrompt, …})` is the only mutating surface; `cmd.on(...)` is observe-only. Mods can add tools, commands, flags, providers, sub-agent types, UI panels/widgets, and answer `onPermissionRequest` — but “the permission prompt … is not on the list and never will be,” and the docs warn plainly: “There is no sandbox. A mod is code that runs with your permissions.” Built-in features (providers, titling, learning, todos) ride the same API. Manage with `cmd mods add/list/update/remove/enable/disable`; kill switches are `cmd --safe-mode` and `mods.disableAll`.

## Official sources

- [Permissions](https://commandcode.ai/docs/permissions)
- [Hooks](https://commandcode.ai/docs/hooks)
- [Mods](https://commandcode.ai/docs/mods)
- [CLM Reference](https://commandcode.ai/docs/reference/cli)
- [Interactive Mode](https://commandcode.ai/docs/interactive-mode)
