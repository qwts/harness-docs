# Sessions and context

**Verified:** 2026-11-16 (Pi v1.5.2)

## Session model

“Pi coordinates model requests, tool execution, context assembly, and session storage. A session is Pi's record of a conversation, including messages, tool calls and results, model changes, compactions, and other events.” The shape is a tree, not a transcript: “Messages and events in a session form a tree. Each path through that tree is a branch. The branch ending at the current entry is the active branch and supplies the history for the next model request.” Files are JSONL under `~/.pi/agent/sessions/--<path>--/<timestamp>_<session-id>.jsonl` (custom IDs via `--session-id` or the SDK; “Session IDs must start and end with a letter or number”), with a session header (`{"type":"session","version":3,...}`) and `id`/`parentId` linking; versions 1–3 exist and “existing sessions are automatically migrated to the current version (v3) when loaded.”

Each session “records the working directory it was created in and never changes it. Pi runs a session in its working directory, so tools, `@path` references, project configuration, and context files resolve from there.” `--continue` and `--resume` differ accordingly: `--continue` uses the current directory; `--resume`, `/resume`, `--session`, and `/import` run “the session's working directory, which can differ from the current directory”; `--fork` works in “the current directory, in a new session.” If the stored directory no longer exists, interactive mode asks to **Continue**, **Fork** (copy “with a new ID, like `--fork`. The original stays unchanged”), or **Cancel** — “Other modes exit with an error.”

## Entry points and commands

CLI: `-c/--continue` (“Continues the most recent session for the current directory”), `-r/--resume` (“Opens the session selector … Press Tab to include sessions from other directories”), `--session <path|id>` (“If the ID only matches sessions stored for another directory, Pi offers to fork it into the current directory instead”), `--session-id <id>` (“opens the current directory's existing session with the ID, or creates it otherwise”), `--fork <path|id>`, `--session-dir <dir>`, `--no-session` (“Uses an in-memory session that is not persisted”), `-n/--name <name>`. Constraints: “`--fork` cannot be combined with `--session`, `--continue`, `--resume`, or `--no-session`”; `--session-id` cannot combine with `--session`/`--continue`/`--resume` but “Combine it with `--fork` to choose the new ID.”

Slash commands: `/new`, `/resume`, `/name [name]`, `/session` (“Show current session information and statistics” — “file, ID, message count, token usage, and cost”), `/tree`, `/fork`, `/clone` (“Duplicate the current session at its current position”), `/compact [instructions]`, `/import <path>` (“Import a JSONL session, preserving its ID and working directory”). “The session picker lets you search, rename, and delete sessions”; deletion from `/resume` uses `Ctrl+D` with confirmation and “When available, pi uses the `trash` CLI to avoid permanent deletion.”

## Branching

| Action | Result | Use it when |
|---|---|---|
| `/tree` | “Moves within the current session file” | “Related alternatives should stay together” |
| `/fork` | “Creates a new session from an earlier user message” | “The alternative should become separate work” |
| `/clone` | “Copies the active branch into a new session” | “You want a separate copy of the current state” |

“In `/tree`, select a user message to put its text back in the editor. Edit and submit it to create another branch.” Leaving a branch can summarize it: “When you leave a branch, Pi can summarize it and attach that summary to the branch you enter” (`branchSummary.reserveTokens` 16384, `branchSummary.skipPrompt`). The storage rule behind all of it: “Continuing from an earlier entry creates another branch in the same file. Forking and cloning copy selected history into a new session file.”

## Compaction

“The model receives the active branch, not every branch in the session file.” Auto-compaction triggers at “`contextTokens > contextWindow - reserveTokens`” with `reserveTokens` 16384 and `keepRecentTokens` 20000 (both configurable per user/project settings and via `compaction.modelOverrides` keyed by exact `provider/modelId`). The pipeline: find the cut point walking backwards until `keepRecentTokens` is reached, summarize with the previous summary as iterative context, “Save `CompactionEntry` with summary and `firstKeptEntryId`,” and rebuild context from summary plus kept messages. “The original entries remain in the session tree,” and on repeated compactions “the summarized span starts at the previous compaction's kept boundary (`firstKeptEntryId`), not at the compaction entry itself.”

Recovery and manual control: “A provider context-overflow error or an early final `stopReason: "length"` can select one compact-and-retry recovery attempt.” “Run `/compact` to compact manually. You can add instructions when the summary should preserve a particular topic or decision.” “Compaction can fail if the provider is unavailable or cannot accept the summarization request. Correct the provider problem and run `/compact` again. Disabling automatic compaction does not disable the manual command.”

## Storage, export, and sharing

“By default, Pi stores sessions under `~/.pi/agent/sessions/`, grouped by working directory. Use `--session-dir`, `PI_CODING_AGENT_SESSION_DIR`, or the `sessionDir` setting to choose another location. The CLI option has highest precedence.” “Use `--no-session` for an ephemeral run. An ephemeral session cannot be resumed after Pi exits.”

“Use `/export` to write the current session as HTML or JSONL. Use `/share` to upload it and get a viewer link. By default, Pi uses a Radius artifact when Radius authentication is configured; otherwise, it uses a private GitHub gist. Set `sessionShareDestination` to `gist` or `radius` to force one destination.” The warning applies to both: “Review exported or shared sessions first. They can contain prompts, model responses, tool arguments, command output, file contents, and extension messages.” `/copy` (or `Ctrl+X`) copies “the last assistant response.” `/bug [description]` prepares “a private report for the Pi developers” — environment and provider configuration “without credential values, plus recorded error diagnostics” — uploaded through `radius.pi.dev` or exported as a zip; “Uploads do not require a login.”

## Steering during a run

| What you want | Action |
|---|---|
| Adjust the current task | “Type a message and press `Enter`” |
| Add work after the current task | “Type a message and press `Alt+Enter`” |
| Return queued messages to the editor | “Press `Alt+Up`” |
| Stop the current task | “Press `Escape`” |

“A message sent with `Enter` waits until the current response and its tool calls finish, then guides the next response. A follow-up sent with `Alt+Enter` waits until Pi finishes the current task. Aborting returns queued messages to the editor.” Queue delivery is configurable (`steeringMode`/`followUpMode`, default `"one-at-a-time"`). Per the agent loop: “Steering messages enter after the current assistant turn. Follow-up messages enter after the agent has finished its pending work.”

## Official sources

- [Sessions and Context](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sessions.md) · [Compaction Reference](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md) · [Session File Format](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/session-format.md) · [How Pi Works](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/how-pi-works.md) · [Use Pi in the terminal](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/usage.md) · [Command Line](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)
