# Claude Code — Runtime Behavior: Hooks, Prompts, Memory, Tool Search (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

## Lifecycle hooks as deterministic kill switches

✅ TRUE — The draft's central hooks claim is correct. `PreToolUse` hooks fire **before** a tool call executes and can **block** it:

- Hook events span the lifecycle: `SessionStart`/`SessionEnd`, `UserPromptSubmit`, `PreToolUse`/`PostToolUse` (plus `PostToolUseFailure`, `PostToolBatch`), `PermissionRequest`, `Stop`, `PreCompact`/`PostCompact`, `SubagentStart`/`SubagentStop`, and more.
- Handlers are shell commands, HTTP endpoints, MCP tool calls, LLM prompts, or subagents; input arrives as JSON (stdin for command hooks).
- A blocking `PreToolUse` decision turns security policy into a **deterministic runtime control** rather than a probabilistic prompt instruction — this framing is accurate.
- ⚠️ PARTIAL: the draft treats hooks as merely "pre-execution barriers"; hooks can also mutate/redirect tool input, add decisions, and react to many other lifecycle points.

## Layered system prompts (CLAUDE.md and friends)

✅ TRUE (with reframing) — Localized Markdown instruction files are **appended/concatenated as an overlay**; they do not replace the base framework instructions. Load order is broadest → most specific: managed policy → user (`~/.claude/CLAUDE.md`) → project (`./CLAUDE.md` or `./.claude/CLAUDE.md`) → local (`./CLAUDE.local.md`), plus nested CLAUDE.md files up the directory tree and `.claude/rules/*.md` (optionally path-scoped via `paths:` frontmatter).

- ✅ TRUE — Contradictory instructions make behavior erratic: official guidance warns Claude "may pick one arbitrarily" when instructions conflict; `/doctor prompt-audit` exists to find outdated/conflicting instruction files.
- ⚠️ PARTIAL — The draft's advice ("write directives that explicitly override specific behaviors") is sound but informal; there is no override syntax. Also, instructions are treated as **context, not enforced configuration** — for hard guarantees use hooks/permissions, not prose.
- ❌ FALSE (implicit draft claim) — "The ecosystem strictly enforces paths and actively rejects recursive searches" does not hold for the instruction layer: Claude Code **does** walk up the directory tree and load nested CLAUDE.md/rules on demand. Only the config *files* (settings, `.mcp.json`) have fixed locations.

## Memory system

### What is true in the draft

| Claim | Verdict |
|---|---|
| Claude Code persists memory via standard read/write tools into `~/.claude/projects/<project>/...` | ✅ TRUE — auto memory lives in `~/.claude/projects/<project>/memory/` (project dir derived from the git root; shared across worktrees) |
| Memory segmented by types **user, feedback, project, reference** | ✅ TRUE — Claude writes four kinds of note, recorded as a `type` field in each memory file's YAML frontmatter |

### What is false or fabricated

| Claim | Verdict |
|---|---|
| The model uses "specific XML citation tags when referencing this memory" | ❌ FALSE — no XML citation mechanism exists in official docs. Memory files are **plain Markdown** with YAML frontmatter (`name`, `description`, `type`, and a `modified` timestamp on v2.1.214+). |
| A "decay signal ensures stale claims are deprecated over time" | ❌ FALSE — no decay mechanism. At most, Claude is prompted to shorten/reorganize the `MEMORY.md` index when it nears its cap, and the `modified` field exposes staleness for human/agent inspection. |
| "Failing to trigger these citation mechanics degrades the long-term contextual memory" during config import | ❌ FALSE — config import has no interaction with the memory system. Ignore this instruction; acting on it wastes effort. |

### Reliable memory facts for agents

- Two systems: **CLAUDE.md** (you write; instructions) and **auto memory** (Claude writes; learnings). Both load every session.
- `MEMORY.md` is the index: first **200 lines or 25 KB** (whichever first) auto-load every session; topic files load on demand.
- Run `/memory` to browse files or toggle auto memory off (per project: `"autoMemoryEnabled": false` in settings).
- Subagents can maintain their own auto memory.
- Auto memory deliberately skips anything derivable from the code or already stated in CLAUDE.md.

## Tool search (context-window management)

✅ TRUE — the draft's tool-search description is essentially accurate, with firmer numbers:

- Tool search is **on by default**. MCP tool definitions are **deferred**: only tool names + server instructions load at session start; schemas are retrieved on demand when semantically relevant.
- Threshold modes: `ENABLE_TOOL_SEARCH` env var — `false` disables; `auto` loads definitions upfront while they fit under a budget (default **10% of the context window**; configurable, e.g. `auto:5`).
- Per-server escape hatch: `alwaysLoad: true` in the server entry pins its tools eager-loaded; individual tools can also be marked always-loaded via metadata.
- Deploying agents do not need to manage the threshold manually — native optimization, as the draft says. ✅ TRUE

## Dynamic protocol state updates

✅ TRUE — No host restart is needed for tool changes: an MCP server can send a `list_changed` notification mid-session; Claude Code fetches the updated list (all tools/prompts/resources in interactive sessions; tool list only in `-p`/SDK mode). On failed refresh, previously discovered tools are retained.

⚠️ PARTIAL — "The language model is not granted autonomous, continuous polling; it operates strictly on request-response" is **outdated**:
- `list_changed` is server-initiated push (v2 runtime holds a stream open for it).
- Claude Code supports opt-in **channels**: a server declaring the `claude/channel` capability (enabled with `--channels`) can push messages into the session (CI results, alerts, chat) so Claude reacts to external events unprompted.
- The approval-gating claim is ✅ TRUE: state-changing tool calls still route through Claude Code's permission system, which presents the payload for user approval.

## Subagent orchestration — retained as GUIDANCE (💬 OPINION)

The following came from blog commentary, not harness specification. It is reasonable engineering guidance but is **not** documented harness behavior; do not cite it as spec:

- 💬 Parallel-don't-sequential: launching subagents in a linear awaited loop hangs the run if one child stalls. Fan out independent tasks concurrently.
- 💬 DAG decomposition: run dependency-free tasks in parallel (isolated worktrees are available), and make abort signals cascade parent→children while a failed child reports back rather than collapsing the whole tree.
- 💬 Terminal states: without explicit completion criteria, agents improvise more actions and burn context. End turns with an explicit completion/blocked/awaiting-input declaration. (Good prompt practice; the harness does not "require" this.)

## Performance & economics — retained as OPINION (💬)

The comparative claims (fewer actions but more tokens per task, faster wall-clock resolution, economics coupled to provider cache-read discounts, "optimized for its native ecosystem") are **blog opinion from a single source** and were not independently verifiable. Treat as one practitioner's assessment of trade-offs, not fact. The underlying design facts it rests on (heavy per-step context, tool-search mitigation) are real; the benchmarks are not.
