# Automation: print, JSON, RPC, SDK

**Verified:** 2026-11-16 (Pi v1.5.2)

## Choosing a mode

“By default, running `pi` opens the interactive terminal interface. When input or output is piped or redirected, Pi uses print mode instead.” All modes “use the same agent, sessions, resources, and tools. The mode determines how input enters Pi, how output is exposed, and whether the process remains available for more commands.”

| Mode | Interface | Lifetime | Use it when |
|---|---|---|---|
| Interactive | Terminal UI | Until the user exits | “A person is working with Pi directly” |
| Print | “Final text on stdout” | One invocation | “A script needs the final assistant response” |
| JSON | “JSONL events on stdout” | One invocation | “A process needs structured progress from a run” |
| RPC | “JSONL commands, responses, and events” | Long-lived | “A process needs bidirectional control” |

“The SDK is not a CLI mode. It embeds the agent directly in a Node.js or Bun process.”

## Print mode

“Print mode runs the supplied prompts, writes the final assistant text to stdout, and exits.” “Print mode writes errors to stderr. A final assistant response with an `error` or `aborted` stop reason produces a nonzero exit status. A response cut off at the model's output token limit (`length` stop reason) still prints its partial text to stdout, writes a warning to stderr, and produces a nonzero exit status.” Non-TTY stdin or stdout selects print mode automatically. Piped stdin is prepended to the first prompt; `@path` arguments attach files; `--` stops option parsing.

## JSON event stream

`pi --mode json "Review this repository" > events.jsonl` — “Pi writes one session header followed by session events, then exits after the supplied prompts finish.” Framing: “The stream uses strict JSONL framing. Each record is one JSON object terminated by LF (`\n`)”, and “Node.js `readline` is not suitable for this stream because it also recognizes those Unicode separators.” “All prompts are supplied when the process starts. The process streams events for that run and then exits; it does not accept later commands.”

Event vocabulary: `agent_start`, `turn_start`, `message_start`/`message_update`/`message_end`, `turn_end`, `agent_end` (“closes one low-level agent run. Automatic retry, overflow recovery, compaction retry, steering, or follow-up work can still continue”), and `agent_settled` (“means Pi has no remaining automatic work for that session-level run”). “A failed or aborted assistant response appears in the event stream but does not by itself produce a nonzero exit status. Inspect the events when success or failure matters.” Streaming reconstruction: “Wire `message_update` records are delta-only” (`text_delta`, `thinking_delta`, `toolcall_delta`, …) — “replace the whole partial message with `message_end.message` when it arrives.” “Stdout is reserved for JSONL. Diagnostics and application logging are written to stderr.”

## RPC mode

“RPC mode keeps Pi running while another process sends commands and receives responses and events” (`pi --mode rpc --no-session`). Commands are JSON objects on stdin; responses and events on stdout, one record per line; “RPC mode rejects `@file` arguments.” “A successful `prompt` response means the prompt was accepted, queued, or handled. It does not mean the run completed. Continue consuming events through `agent_settled` when completion matters.” “RPC commands can change models, inspect state, manage sessions, run shell commands, and answer extension UI requests”; “Extension dialogs form a request-response subprotocol.” For Node.js, “prefer `RpcClient` from `@earendil-works/pi-coding-agent`”: it “starts a Pi RPC child process, correlates requests, exposes typed command methods, and delivers session events to listeners,” and `promptAndWait()` “installs its event listener before sending the prompt, avoiding a race with fast completions.”

## TypeScript SDK

“`@earendil-works/pi-coding-agent` embeds Pi in a Node.js or Bun process. It provides direct TypeScript access to the agent, sessions, tools, models, and resources used by the command-line application.”

```typescript
import { createAgentSession } from "@earendil-works/pi-coding-agent";
const { session } = await createAgentSession();
await session.prompt("What files are in the current directory?");
console.log(session.getLastAssistantText());
session.dispose();
```

Lifecycle and prompting: “`createAgentSession()` creates an `AgentSession`. The session owns one conversation, its model and tools, queued messages, compaction state, and extension runtime.” “`prompt()` handles extension commands and expands file-based prompt templates before ordinary user messages enter the agent … A prompt sent while the session is already streaming must specify whether it should steer the current run or follow it. Calling `prompt()` without that choice rejects rather than guessing.” `steer()`/`followUp()` return `"queued"` or `"handled"`; `abort()` “stops the active operation and waits for the session to become idle.” Subscribe before prompting; “Use `agent_settled` when the host needs to know that Pi will not continue automatically.”

Storage and composition: “`SessionManager` owns the persisted or in-memory entry tree and tracks its active leaf … `SessionManager` is authoritative for finalized model context.” In-memory sessions use `SessionManager.inMemory()`. Each boundary is injectable — `modelRuntime`, `model`, `thinkingLevel`, `scopedModels`, `settingsManager`, `sessionManager`, `resourceLoader`, `tools`/`noTools`/`excludeTools`/`customTools`. `AgentSessionRuntime` “adds `newSession()`, `switchSession()`, `fork()`, and `importFromJsonl(). Each operation replaces the active `AgentSession` … After a runtime replacement, subscriptions belong to the old `AgentSession` and must be rebound.” SDK sessions “do not load built-in extensions” — add `builtin:mcp`, codemode, and tool-search to the resource loader explicitly.

## Rebranding and extension flags

“A source fork can change the CLI name and configuration directory through `package.json`” — a `piConfig` object with `name` and `configDir` — “These settings affect the CLI banner, configuration paths, and derived environment variable names.” “The top-level help also includes options registered by loaded extensions. … Extensions may register additional long-form options. Unknown short options are rejected.”

## Scheduling, wakeups, and telemetry

No scheduler, cron, daemon, or wakeup surface exists in the documentation — the long-running integration surfaces are RPC mode and the SDK, and every CLI mode is tied to a process Pi runs. Update and network controls: `pi update` (self / `--extensions` / `--models` / `--all`), `PI_SKIP_VERSION_CHECK`, `PI_OFFLINE` (“Disable automatic network activity, including model catalog refreshes”), `enableInstallTelemetry` (default true; “Does not control update checks”), `enableAnalytics` (default false), and `PI_TELEMETRY`.

## Official sources

- [CLI Integration](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli-integration.md) · [JSON Event Stream](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/json.md) · [RPC Protocol](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/rpc.md) · [SDK](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sdk.md) · [Environment Variables](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/environment-variables.md) · [Command Line](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)
