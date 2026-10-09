# App Server & Runtime Behavior (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against the official `app-server`, `codex/cli`, `cli-customization`, and `open-source` pages (learn.chatgpt.com) plus independent protocol coverage.

## What the App Server is

✅ TRUE — `codex app-server` is a long-lived Rust process (open source at `openai/codex/codex-rs/app-server`) hosting the agent loop, thread lifecycle, model targeting, sandboxed tool execution, MCP, and approvals. The VS Code extension and other rich clients are powered by it.
✅ TRUE — external orchestrators and internal GUIs integrate via structured JSON-RPC 2.0 instead of scraping terminal output; configuration parsing, MCP, and sandboxing behave consistently across surfaces.
⚠️ PARTIAL — transports are stdio (default) and WebSocket **plus a Unix-socket transport** (`--listen unix://`) the draft omitted. WebSocket is **experimental and unsupported for production workloads**.
✅ TRUE (surprising) — the `"jsonrpc":"2.0"` header is explicitly omitted on the wire (stated verbatim in official docs).

## Handshake

✅ TRUE — the client must send `initialize` first (with `params.clientInfo`: name, title, version); requests before initialization are rejected ("Not initialized"); repeated `initialize` returns "Already initialized".
⚠️ PARTIAL — the draft reverses the handshake: after the server's `initialize` response, the **client** emits the `initialized` notification — the server does not send it.
✅ TRUE — `capabilities.experimentalApi = true` unlocks experimental methods/fields (`thread/turns/list`, `thread/items/list`, background terminals, `process/*`, `environment/info`, …). `optOutNotificationMethods` suppresses specific notifications.

## WebSocket ingress limits

✅ TRUE — `codex app-server --listen ws://127.0.0.1:4500` is the documented example. WebSocket mode uses bounded queues; when ingress is full the server rejects requests with JSON-RPC error `-32001` / "Server overloaded; retry later."; clients should retry with exponentially increasing delay and jitter. (Confirmed verbatim.)

## Thread / Turn / Item

✅ TRUE — the protocol is organized around three nested primitives:
- **Thread** — durable conversation container (persisted JSONL rollout log, SQLite-backed metadata incl. gitInfo). Pause/resume/fork/archive map to real methods: `thread/start`, `thread/resume`, `thread/fork` (with `lastTurnId`/`ephemeral`), `thread/archive`/`unarchive`, `thread/read`, `thread/list`, `thread/delete`, `thread/unsubscribe` (auto-unload after an inactivity grace period).
- **Turn** — a single user request plus the agent work that follows: `turn/start` (bound to threadId), `turn/steer`, `turn/interrupt`; ends with `turn/completed`.
- **Item** — atomic input/output units (user/agent messages, command runs, file changes, tool calls) streamed via `thread/started`, `turn/started`, `item/started`, `item/completed`, `item/agentMessage/delta`. The draft's "text deltas, command execution requests, and file system patches" is correct.

## Approvals (server-to-client requests)

✅ TRUE — when settings require approval, the app-server sends a **server-initiated JSON-RPC request** and the client responds with a decision payload; the turn halts until decided. Interception requires correlating the request id and answering with the decision.
⚠️ PARTIAL — `execCommandApproval` and `applyPatchApproval` are real but **deprecated legacy** methods (legacy SendUserTurn/SendUserMessage path). Current versions use `item/commandExecution/requestApproval`, `item/fileChange/requestApproval`, and `item/permissions/requestApproval` (plus `mcpServer/elicitation/request` and `tool/requestUserInput`).
⚠️ PARTIAL — exact response enums vary by method/version: current item/* approvals reply with `accept | acceptForSession | decline | cancel`; older methods used a simpler decision payload (the draft's "decision": "accept"/"decline" is the legacy shape).
⚠️ PARTIAL — "fails to intercept → the loop stalls indefinitely, deadlocking the thread": the turn does block awaiting a decision, but current app-server **refuses unknown server→client requests with -32601** rather than stalling, and in non-interactive flows an action needing fresh approval **fails and surfaces the error back to the parent workflow** instead of deadlocking.

## Schema generation

✅ TRUE — `codex app-server generate-ts` and `codex app-server generate-json-schema` emit version-pinned artifacts ("each output is specific to the Codex version you ran"). Regenerate after every CLI upgrade to catch protocol changes; pinned artifacts prevent schema drift for a given binary.

## CLI surface

✅ TRUE — terminal-native client, installed via standalone installers (curl / PowerShell), npm (`@openai/codex`), or Homebrew (`brew install --cask codex`). The Rust binary is open-sourced under **Apache-2.0** (openai/codex LICENSE).
✅ TRUE — headless automation: `codex exec` (`codex e`) for scripted/CI runs; prompts can be piped via stdin; `--json` + `--output-last-message` for machine-readable output. Config and the host shell environment are inherited.
✅ TRUE — vim-style modal editing (incl. Vim R replace mode), mouse selection/copy (smart copy picker), and in-terminal Mermaid + TeX rendering are confirmed by multiple independent release-based sources. Official `cli-customization` documents themes, syntax-highlighted diffs, shell completions, and the Ctrl+G prompt editor (it does not enumerate the vim/mermaid features).
💬 OPINION — "the CLI is the primary integration target because its version can be pinned in lockfiles, guaranteeing absolute reproducibility": npm/brew version pinning is real and sensible for CI; "absolute" is editorial.

## Desktop app surface

✅ TRUE — the ChatGPT desktop app is a **closed-source** client (absent from the official open-source components list, which covers CLI, SDK, app-server, skills, plugins) and updates on OpenAI's schedule (enterprises can manage desktop-app updates via deployment tooling).
✅ TRUE — `/app` in an active CLI session hands the session off to the desktop app (shipped ~v0.138; note known issue openai/codex#31709: desktop may not refresh session context immediately after handoff).
✅ TRUE — the desktop app surfaces subagent threads, approvals, and diff review — the draft's "human supervision of multi-agent workloads, parallel task management, and complex diff reviews" is accurate.
💬 OPINION — "highly effective for project managers overseeing multi-threaded code generation across various repositories" is editorial framing of the capabilities above.
