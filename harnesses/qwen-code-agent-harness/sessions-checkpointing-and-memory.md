# Sessions, checkpointing, and memory

**Verified:** 2026-11-16 (Qwen Code 0.27.4 npm latest; docs site main)

## Sessions

“Session data is project-scoped JSONL under `~/.qwen/projects/<sanitized-cwd>/chats`.” Sessions carry “conversation history, tool outputs, and chat-compression checkpoints.”

- **Resume:** `--continue` resumes “the most recent session for this project”; `--resume <sessionId>` opens a specific session (“or choose interactively”). Interactive: `/resume` (alias `/continue`). Worktree sessions re-attach: “`qwen --resume <session-id> --worktree=my-feature` … re-attaches to the existing dir.”
- **Listing:** `qwen sessions list` (“List recent conversation sessions”, default limit 20, `--json` emits JSON Lines with `sessionId, startTime, mtime, prompt, gitBranch, customTitle, titleSource, filePath, cwd`); `qwen sessions ps` shows “interactive sessions running right now” — “Records left behind by a killed session are swept from the registry as they are found. A managed record is persistent bookkeeping, not evidence that its worker is still alive. A one-shot `qwen -p` run never registers.”
- **In-session commands:** `/compress` (“Replace chat history with summary”), `/compress-fast` (“Fast compression without AI — strips old tool outputs and thinking parts”), `/branch` (“Fork the current conversation into a new session”), `/fork <directive>` (“Spawn a background agent that inherits the full conversation”), `/rewind` (alias `/rollback`), `/restore`, `/delete`, `/export` (`html`/`md`/`json`/`jsonl`), `/rename`/`/tag`, `/summary`, `/recap` (“Generate a one-line session recap now”), `/clear` (aliases `/reset`, `/new`), `/pr` (“Bind a GitHub pull request to this session”), `/commit` (“Create a git commit with an AI-drafted message”).
- **Writer locks:** “Updated daemons can share Conversations and use different sessions at the same time. A loaded session still has one writer.” `session_writer_conflict` “means the writer fence prevented access … It is not proof that another writer is currently alive.” Recovery “fails closed”: “Malformed records, uncertain transcript identity, and residual transition claims fail closed. Elapsed time alone never authorizes takeover.”
- **Runtime locations:** `QWEN_RUNTIME_DIR` overrides runtime output (session records, logs, todos); `QWEN_CONVERSATIONS_DIR` selects the daemon's shared Conversations root (“Defaults to `~/Documents/Qwen Code/Conversations`”; “Fixed for the daemon lifetime”).

## Checkpointing

“Qwen Code includes a Checkpointing feature that automatically saves a snapshot of your project's state before any file modifications are made by AI-powered tools.” Each checkpoint captures “**A Git Snapshot:** A commit … in a special, shadow Git repository located in your home directory (`~/.qwen/history/<project_hash>`) … It does **not** interfere with your own project's Git repository”, plus “**Conversation History**” and “**The Tool Call**” (JSON under “`~/.qwen/tmp/<project_hash>/checkpoints`”). “The Checkpointing feature is disabled by default” — enable per-run with `--checkpointing` or set `"general": {"checkpointing": {"enabled": true}}`. `/restore` will “Revert all files in your project to the state captured in the snapshot,” “Restore the conversation history in the CLI,” and “Re-propose the original tool call.” Under the Linux execution sandbox, “file checkpoints … are disabled or rejected,” and “Conversation-only rewind remains available.”

## QWEN.md context

Three instruction files load per session: “`~/.qwen/QWEN.md`” (you, all projects), “`QWEN.md` in the project root” (team, committed), “`.qwen/QWEN.local.md`” (personal, per project; “It loads **after** the shared project `QWEN.md`”). Hierarchical loading scans “the current working directory and then in each parent directory up to either the project root (identified by a `.git` folder) or your home directory.” “If your repository already has an `AGENTS.md` file … Qwen reads that too.” Imports: `@path/to/file`. `/memory` opens “the memory management dialog”; `/init` generates a starter QWEN.md.

## Auto-memory (managed)

“**Auto-memory** — notes Qwen writes itself based on what it learns from you.” “Auto-memory runs in the background. After each of your conversations, Qwen quietly saves useful things it learned.” It looks for four kinds of things worth remembering — for example “Your role, background, how you like to work” and “Corrections you made, approaches you confirmed.” Channel-relayed turns are excluded — “When a turn that answers a message relayed by a channel … ends, Qwen runs no extraction and schedules no periodic cleanup: the sender is often not you”; “a reply to a channel message can draw on what is already saved” and “A channel chat keeps durable context in channel memory instead.”

Settings: `enableManagedAutoMemory: true`, `enableManagedAutoDream: true` (“automatic consolidation (deduplication and cleanup)”), `enableAutoSkill: false` (“background review for reusable project skills after tool-heavy sessions”; `autoSkillConfirm: true` asks before saving), `enableTeamMemory: false` (git-tracked `.qwen/team-memory/`, secret-scanned writes; `enableTeamMemorySync` commits/pulls/pushes at session start), `enableStructuredRecall: false` (switches recall “from the flat `MEMORY.md` listing to the structured protocol (hierarchical memory tree, focused subtree, `search_memory` tool)”). Background memory agents are bounded by `agentTimeoutMinutes`/`agentMaxTurns`. Commands: `/memory`, `/remember`, `/dream`.

Headless note: after printing its output, “the CLI normally waits for tracked extraction, dreaming, and scheduled skill-review tasks before finishing the invocation. A one-shot process may therefore remain running after its output is written,” with a 5000 ms final grace period.

## External memory

“**External memory (`context_search`)** — an optional MCP tool Qwen calls on demand to search Mem0 or a local session index.” Mem0 is configured via `memory.mem0` in user or system settings — “Its `envKey` references a credential variable (default `MEM0_API_KEY`) … the top-level `env` field can supply the value without a shell export … No separate npm package, MCP registration, or manually configured write-confirmation Hook is needed.”

## Cross-session protocol and messaging

“`agents.crossSessionMessaging` … Let Qwen Code sessions on this machine send each other messages over a per-session local socket. On by default.” Inbound behavior is set by `agents.crossSessionInbound` (`accept`/`hold`/`refuse`; unset means “user-minted controllers and this session's own child processes auto-deliver, while other sessions use review-class parity; other messages are held for review”). `qwen sessions controllers add --label <name>` manages trusted controller tokens. “Interactive sessions remain listed by `qwen sessions ps` without a messaging address; daemon- or editor-hosted sessions register only while messaging is active.” The feature is disabled by `--bare` and `--safe-mode` (“which keep this session out of messaging whatever it says”). A workspace may set it to `false` only.

## Official sources

- [Conversations recovery](https://qwenlm.github.io/qwen-code-docs/en/users/conversations-recovery/) · [Checkpointing](https://qwenlm.github.io/qwen-code-docs/en/users/features/checkpointing/) · [Memory](https://qwenlm.github.io/qwen-code-docs/en/users/features/memory/) · [Mem0](https://qwenlm.github.io/qwen-code-docs/en/users/features/mem0/) · [Session overview](https://qwenlm.github.io/qwen-code-docs/en/users/features/session-overview/) · [Cross-session protocol](https://qwenlm.github.io/qwen-code-docs/en/users/features/cross-session-protocol/) · [Commands](https://qwenlm.github.io/qwen-code-docs/en/users/features/commands/)
