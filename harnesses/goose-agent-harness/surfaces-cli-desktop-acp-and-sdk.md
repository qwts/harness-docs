# CLI, Desktop, ACP, and SDK surfaces

**Verified:** 2026-10-29

## Surface matrix

| Surface | Role | Important distinction |
|---|---|---|
| goose Desktop | Native macOS/Linux/Windows app and ACP client | Starts a local `goose serve` backend by default; can route all backend requests to a remote server |
| Interactive CLI | Terminal sessions and session management | Shares core config with Desktop; has its own commands, display, and session controls |
| `goose run` | Headless/one-shot and recipe automation | Starts a session, executes input, and exits unless interactive mode is requested |
| `goose acp` | ACP agent over stdio | An external client owns the subprocess and communicates over stdin/stdout |
| `goose serve` | ACP agent over HTTP/WebSocket | Authenticated `/acp` endpoint; local by default, remotely deployable with explicit network/TLS settings |
| ACP provider | External ACP agent used as goose’s model/agent provider | Opposite direction from exposing goose as an ACP agent; extensions pass through, but resume/fork are unavailable |
| Editor integrations | ACP clients such as Zed or experimental VS Code extension | Editor context and lifecycle are supplied by the integration, not automatically by every ACP client |
| GDK/SDK | Native Rust crates and shared provider bindings | Python/Kotlin bindings do not expose the full native agent orchestration surface |
| Roaming | Experimental peer-to-peer ACP transport | Source build with `roaming` feature required; absent from released binaries |

Shared core configuration does not imply feature parity across these surfaces.

## Desktop

Desktop normally starts its own `goose serve` process on an available loopback port and connects to `/acp` over WebSocket. It supplies visual recipe library/editing, scheduling, extension management, MCP App windows, session-specific permission controls, and settings UI.

Changing a default permission mode affects new sessions; existing sessions retain their mode. Provider changes can refresh configuration/inventory without an app restart, but active chats retain their original provider instance.

When configured with an external goose server, Desktop routes all backend requests to that remote process. Execution then occurs in the server’s environment, filesystem, credentials, and working context—not merely in the local UI’s environment.

## Interactive and headless CLI

`goose session` starts or resumes interactive sessions. Current session storage uses a SQLite `sessions.db`; legacy JSONL files are imported but no longer managed. Resume, fork, edit-and-fork, export, naming, filtering, and removal are CLI session operations, subject to provider limitations.

`goose run` accepts text (`-t`), an instruction file (`-i`), stdin (`-i -`), or a recipe. By default it starts a new session, performs the task, and exits. `--interactive` continues into an interactive session afterward.

Useful automation distinctions:

- `--no-session` discards the session record/output path; it does not make tool execution read-only or undo side effects.
- `--output-format json` returns a complete structured result; `stream-json` emits events during execution.
- JSON output and process completion are not proof of task correctness; use deterministic checks or recipe response/retry validation.
- `--provider` and `--model` override environment provider/model settings for the run.
- `--with-extension`, `--with-builtin`, and `--with-streamable-http-extension` apply to that session without persistent installation.
- `--debug` can expose complete tool responses, detailed parameters, and full paths; handle logs as potentially sensitive.

## ACP agent over stdio, HTTP, and WebSocket

An ACP client can launch `goose acp` as a subprocess. The client owns the process lifetime and exchanges ACP messages over stdin/stdout.

For HTTP/WebSocket, run `goose serve`. Direct startup defaults to `127.0.0.1:3284` and exposes `/acp`. `GOOSE_SERVER__SECRET_KEY` configures shared-secret authentication:

- HTTP clients use `X-Secret-Key`.
- WebSocket clients can use the same header.
- Browser WebSocket clients use the `?token=` query parameter.
- Missing or incorrect credentials receive `401 Unauthorized`.

`--dangerously-unauthenticated` disables authentication and is appropriate only when untrusted traffic cannot reach the server. Non-loopback browser origins must be explicitly allowed; specifying `--allowed-origin` replaces default loopback origins, and scheme/port matching is exact.

### Remote server

For remote Desktop use, bind the server deliberately, configure a strong shared secret, and enable TLS. Desktop accepts HTTP and HTTPS, but HTTP is not recommended across a network. Certificate fingerprint pinning requires HTTPS; if no fingerprint is supplied, Desktop uses trust-on-first-use for the first certificate seen.

Binding to `0.0.0.0` exposes the listener beyond loopback and makes firewall, authentication, TLS, and secret handling operational requirements. The shared secret is not synchronized automatically between server and Desktop.

## Editor and client examples

Zed can install goose from the ACP Registry or launch a custom `goose acp` command. Registry installation reads existing goose provider, model, and extension configuration. Zed `context_servers` using stdio or HTTP become available to goose; deprecated SSE servers do not. On a name collision, goose’s own extension configuration wins.

The experimental goose for VS Code extension also uses ACP. It adds selected-code and `@` file context chips, streaming chat, enabled-extension access, and history replay. Those editor-specific affordances should not be generalized to every ACP client.

## ACP providers and deprecated CLI providers

ACP provider support lets goose use another agent as its provider while passing goose extensions through as MCP servers. Current limitations include no goose session resume/fork and distinct ACP versus goose session IDs. This is separate from goose acting as an ACP server for Zed, Desktop, or a custom client.

Deprecated CLI providers wrap external tools such as Claude Code and Codex. They filter goose extension information and rely on the external tool’s own tools and native permission/sandbox mappings. For example, Codex CLI mode flags are external Codex behavior, not evidence of a general goose-core OS sandbox. Prefer current ACP providers where documented.

## Roaming agents

Roaming provides authenticated peer-to-peer ACP transport over iroh/QUIC, directly or through relays. It is experimental and requires a source build with `--features roaming`; released binaries do not include `goose roam`.

Peers exchange non-secret connection cards and must accept the relevant public key. A card alone grants no access. The host’s accepted-peer policy controls connection, and comparing fingerprints out of band is recommended.

The remote side carries ordinary ACP session operations. Each connecting client gets its own agent and sessions; simultaneous co-driving of one live session is not part of the current model. A shared host agent runs in the directory where `goose roam share` started unless `--cwd` overrides it. `roam bridge` proxies ACP to a local client without running another agent.

## GDK and SDK scope

Native Rust crates expose agent state/tools, providers, and context management. The `goose-sdk` shared bindings generated with UniFFI expose provider construction, streaming/non-streaming completions, tool/reasoning chunks, conversation compaction, declarative providers, and request logs.

- Rust can use the native crates or shared bindings.
- Python installs `goose-sdk` and imports `goose`.
- Kotlin/JVM uses the `io.github.aaif-goose:gdk` artifact and `io.github.aaif_goose` package.
- The SDK is alpha; pin exact versions and match documentation to the selected release.

The shared Python/Kotlin API is a **provider-layer binding**, not a promise of full goose agent orchestration, extension hosting, recipes, schedules, or ACP server functionality.

## Custom distributions

The upstream project documents custom distributions with preconfigured providers, bundled MCP tools, branding, default behavior, recipes, and new ACP-based user interfaces. A distribution can alter defaults and supply executable components, so its provenance and bundled configuration must be reviewed independently of upstream defaults.

## Official sources

- [Installation](https://goose-docs.ai/docs/getting-started/installation)
- [Running tasks](https://goose-docs.ai/docs/guides/running-tasks)
- [CLI commands](https://goose-docs.ai/docs/guides/goose-cli-commands)
- [ACP agent](https://goose-docs.ai/docs/gdk/acp)
- [Remote goose server](https://goose-docs.ai/docs/guides/remote-goose-server)
- [Zed integration](https://goose-docs.ai/docs/gdk/acp/zed)
- [Experimental VS Code extension](https://goose-docs.ai/docs/experimental/vs-code-extension)
- [ACP providers](https://goose-docs.ai/docs/guides/acp-providers)
- [Deprecated CLI providers](https://goose-docs.ai/docs/guides/cli-providers)
- [Roaming agents](https://goose-docs.ai/docs/guides/roaming-agents)
- [SDK](https://goose-docs.ai/docs/gdk/sdk)
- [Custom distributions](https://github.com/aaif-goose/goose/blob/main/CUSTOM_DISTROS.md)
