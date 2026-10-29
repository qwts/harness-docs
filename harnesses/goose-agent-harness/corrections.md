# Goose corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| Goose is only an interactive terminal coding agent. | ❌ FALSE | Current surfaces include native Desktop, interactive and headless CLI, recipes/schedules, ACP server and client roles, editor integrations, roaming, and SDKs. |
| The current upstream repository is `block/goose`. | ❌ FALSE | The current repository is `aaif-goose/goose`; the project is part of the Agentic AI Foundation at the Linux Foundation. |
| Desktop and CLI maintain separate core provider and extension settings. | ❌ FALSE | They share core configuration, although interface state, active sessions, some credentials, and some features remain surface-specific. |
| Saving a new provider immediately replaces the provider in every active chat. | ❌ FALSE | Configuration and inventory can refresh, but an active chat retains the provider instance with which it started. |
| Environment variables outrank every explicit command and recipe setting. | ❌ FALSE | They outrank config-file defaults generally, while documented CLI run flags, recipe settings, and subagent-specific rules add their own precedence. |
| Provider API keys belong in `config.yaml`. | ❌ FALSE | Goose ignores provider keys there; use the provider environment, system keyring, or supported file-based secret storage. |
| `secrets.yaml` is an encrypted secret store. | ❌ FALSE | File-based secret storage is plaintext; the system keyring is preferred when available. |
| Every provider type exposes the same goose tools and session operations. | ❌ FALSE | Standard API/local, ACP, and deprecated CLI providers have different extension, permission, resume, and fork behavior. |
| ACP providers support goose session resume and fork. | ❌ FALSE | Current ACP-provider limitations explicitly exclude both operations. |
| Pass-through CLI providers are the recommended way to use Claude and Codex. | ❌ FALSE | Those providers are deprecated for backward compatibility; current documentation recommends ACP providers. |
| CLI providers give the external CLI access to goose’s full MCP extension ecosystem. | ❌ FALSE | They filter goose extension information and use the external tool’s own tool ecosystem. |
| The tool shim uses the main conversation model to repair its own tool calls. | ❌ FALSE | It uses a separate interpreter model, through Ollama by default or a configured local backend. |
| Goose reads only a root `.goosehints` file. | ❌ FALSE | It supports global and hierarchical local context, with `AGENTS.md` and `.goosehints` discovered by default. |
| Every nested instruction file is loaded at session startup. | ❌ FALSE | Additional nested files can be discovered as goose accesses deeper directories and then remain active for that session. |
| `@file` and a plain file reference have identical context behavior. | ❌ FALSE | `@file` includes content immediately; a plain reference directs goose to consult the file when needed. |
| Persistent MOIM instructions are deterministic guardrails that cannot be bypassed. | ❌ FALSE | They are model-observed context injected each turn, not an enforcement boundary; use actual tool/process controls for hard restrictions. |
| Goose asks before file changes by default. | ❌ FALSE | Autonomous mode is the default, allowing changes and extension use without approval. |
| Smart Approval deterministically parses every command and its effects. | ❌ FALSE | Read/write classification is best effort and interpreted by the configured LLM provider. |
| Manual Approval necessarily prompts for every read-only tool call. | ❌ FALSE | The documented approval flow focuses on tools classified as writes and supports per-tool overrides. |
| Autonomous Developer tools are confined to the current project by a core OS sandbox. | ❌ FALSE | They can execute commands and modify files with the user’s privileges and access; no general core OS sandbox is documented for this path. |
| Per-tool permissions are the same as enabling or disabling extension tools. | ❌ FALSE | Approval levels govern invocation; enabled extensions, `available_tools`, provider type, and recipe/subagent setup separately govern availability. |
| `available_tools` restricts what an allowed tool or program can do internally. | ❌ FALSE | It filters which extension tools are loaded; it does not sandbox the behavior of the remaining tools. |
| Prompt-injection detection is enabled by default. | ❌ FALSE | It is opt-in through configuration or Desktop settings. |
| Prompt-injection detection guarantees malicious commands cannot run. | ❌ FALSE | It is a fallible safeguard; findings can be allowed, and allowed commands run with full user permissions. |
| ML-based injection classification keeps all content on the local machine. | ❌ FALSE | Tool-call content and recent messages are sent to the configured classification endpoint. |
| Adversary review fails closed when its reviewer errors. | ❌ FALSE | Reviewer failure allows the tool call. |
| Adversary mode reviews every tool by default. | ❌ FALSE | It reviews `shell` by default; additional tools require a `tools:` selection. |
| Any hook event can block its associated action. | ❌ FALSE | `PreToolUse` can deny tool calls and `Stop` can continue a turn; most other events are observational. |
| A broken `PreToolUse` policy hook blocks the tool by default. | ❌ FALSE | Runtime hook failure defaults to allow; set `on_failure: block` for that action. |
| `on_failure: block` also makes malformed hook configuration fail closed. | ❌ FALSE | It governs runtime failures only, not all configuration-loading or matcher errors. |
| `PreToolUseResult` is a durable audit log. | ❌ FALSE | Delivery is awaited but best-effort and not durable; absent or failed subscribers leave no delivery record. |
| Subagent process/context isolation is a filesystem and network sandbox. | ❌ FALSE | Subagents are temporary independent executions whose inherited or configured tools retain their actual access. |
| Subagents can autonomously launch in every permission mode. | ❌ FALSE | Automatic creation occurs in autonomous mode; they are disabled in Manual, Smart, and Chat Only modes. |
| A subagent can spawn another subagent and change parent extensions or schedules. | ❌ FALSE | Nested spawning, extension management, and schedule management are blocked for subagents. |
| A failed subagent always returns a partial result. | ❌ FALSE | Failure or timeout returns no output for that subagent; parallel runs return only successful results. |
| A custom agent file defines a dedicated MCP server list and scheduled workflow. | ❌ FALSE | It defines role/instructions and optional model preference; available session extensions and recipes provide those other capabilities. |
| Loading an agent and delegating to it are identical. | ❌ FALSE | Loading adds instructions to the current conversation; delegation runs a separate session and returns a result. |
| Every skill’s full contents are injected at session startup. | ❌ FALSE | Names/descriptions are discovered at startup; full instructions load on match or explicit request. |
| Skill supporting scripts execute automatically when the skill loads. | ❌ FALSE | Loading supplies instructions/resources; later script execution occurs through available tools with their real privileges. |
| Goose plugins are simply MCP server packages. | ❌ FALSE | Current plugins package skills, hooks, or both; MCP extensions are a separate extension mechanism. |
| Manually copied plugins and all installed plugins support managed auto-update. | ❌ FALSE | Auto-update applies to eligible Git-backed plugins installed through the plugin command, not arbitrary manually copied directories. |
| Disabling a plugin only hides its skills while leaving hooks active. | ❌ FALSE | Disabled plugins are skipped during discovery, so neither their skills nor hooks load. |
| The CLI accepts `.yml` recipe files interchangeably with `.yaml`. | ❌ FALSE | Desktop accepts `.yml`; the CLI supports `.yaml` and `.json`. |
| A recipe is only a saved prompt. | ❌ FALSE | It can package instructions, parameters, extensions, provider/model settings, structured output, retries, and subrecipes. |
| A recipe’s explicit extension list is merged with every default platform extension. | ❌ FALSE | Only listed extensions are available; delegation requires `summon` unless subrecipes cause its automatic injection. |
| Sharing a recipe includes the author’s credentials and memory. | ❌ FALSE | Recipes exclude global/local memory, API keys/personal credentials, and system-level settings. |
| Recipe retry resets all filesystem and external side effects. | ❌ FALSE | Retry resets agent message history, not files, processes, network calls, or other side effects. |
| Schema-valid structured output proves the task completed correctly. | ❌ FALSE | It validates response shape; deterministic success checks must validate actual artifacts and effects. |
| Desktop Recipe Library automatically lists every recipe file discoverable by the CLI. | ❌ FALSE | Desktop lists explicitly saved or imported recipes; CLI discovery scans configured filesystem and repository sources. |
| Subrecipe sessions share the parent’s history and state. | ❌ FALSE | They run in separate sessions without shared conversation history, memory, or state, though external side effects still persist. |
| Subrecipes can nest recursively. | ❌ FALSE | Subrecipes cannot define their own subrecipes. |
| A slash command mapped to a missing recipe reliably returns a command error. | ❌ FALSE | Missing or invalid recipe mappings are treated as ordinary text sent to the model. |
| Recipe schedules are necessarily vendor-hosted cloud jobs. | ❌ FALSE | The documented scheduler is a goose Desktop/CLI surface; execution environment and privileges must be assessed where goose runs. |
| Trusting a recipe once suppresses review even after its contents change. | ❌ FALSE | Desktop warns again when the recipe changes. |
| Enabling an extension mid-session changes the default for future sessions. | ❌ FALSE | Mid-session and dynamic enablement apply only to the current session unless defaults are changed separately. |
| CLI `--with-extension` permanently installs the extension. | ❌ FALSE | It enables the extension for that launched session without persistent installation. |
| An extension allowlist constrains all behavior of an approved MCP server. | ❌ FALSE | It matches installation commands; approved tools retain their actual capabilities. |
| Known-malware scanning proves an extension is safe. | ❌ FALSE | It can block detected malicious packages but is not a complete provenance or behavioral audit. |
| Code Mode JavaScript has direct filesystem, shell, network, and environment access. | ❌ FALSE | The fresh Boa context lacks those direct capabilities and accesses effects only through enabled tools. |
| Code Mode bypasses normal tool permissions because calls originate in JavaScript. | ❌ FALSE | Tool calls retain goose’s extension lease, permission flow, cancellation, and mutation handling. |
| MCP Apps’ sandboxed windows sandbox the agent and Developer shell too. | ❌ FALSE | The sandbox belongs to the standalone app UI, not the creating agent or its other tools. |
| Desktop always executes agent work on the local machine. | ❌ FALSE | It can route all backend requests to an external `goose serve` process, shifting execution to that environment. |
| `goose run --no-session` makes execution read-only. | ❌ FALSE | It discards session storage/output state but does not prevent tool side effects. |
| JSON output or normal process completion proves a headless task succeeded. | ❌ FALSE | Output format and completion are not task validation; use deterministic checks and inspect effects. |
| `goose serve` is unauthenticated by default when a secret is expected. | ❌ FALSE | `GOOSE_SERVER__SECRET_KEY` configures required shared-secret authentication; unauthenticated mode requires an explicit dangerous flag. |
| `--dangerously-unauthenticated` is safe for any network-exposed server. | ❌ FALSE | It is documented only for servers isolated from untrusted traffic. |
| Adding `--allowed-origin` preserves all default loopback origins automatically. | ❌ FALSE | Supplying allowed origins replaces the defaults, so every required loopback and deployed origin must be listed. |
| ACP provider support and exposing goose as an ACP agent are the same direction of integration. | ❌ FALSE | One uses an external agent as goose’s provider; the other lets external clients drive goose’s runtime. |
| Every ACP editor automatically has the same selections, context chips, and history UI. | ❌ FALSE | Those capabilities depend on the client/integration, as shown by the distinct Zed and experimental VS Code setups. |
| Zed can pass deprecated SSE MCP servers to goose. | ❌ FALSE | Zed-provided stdio and HTTP servers are supported; deprecated SSE is not. |
| A Zed `context_servers` entry overrides a same-named goose extension. | ❌ FALSE | Goose uses its own extension configuration on a name collision. |
| Roaming is included in released goose binaries. | ❌ FALSE | It is experimental and requires a source build with the `roaming` feature. |
| Possession of a roaming connection card grants access. | ❌ FALSE | Cards contain public reachability/identity information; the host must accept the peer key. |
| Roaming lets multiple clients co-drive one live session simultaneously. | ❌ FALSE | Each client gets its own agent and sessions; co-driving is not part of the current model. |
| Python and Kotlin shared bindings expose the complete goose agent runtime. | ❌ FALSE | Their documented shared surface is the provider layer; broader native agent orchestration is in Rust crates. |
| The SDK has a stable API across all releases. | ❌ FALSE | It is alpha, and exact version pinning plus matching documentation is recommended. |
| Anonymous usage collection is enabled without asking. | ❌ FALSE | Goose asks for permission on first use, and the setting can be changed. |
| Opt-in usage metrics include conversation contents and tool arguments. | ❌ FALSE | The documented anonymous metrics exclude conversations, code, tool arguments, detailed errors, and personal data. |
| Using a local model guarantees no prompt or tool data leaves the machine. | ❌ FALSE | Remote MCP extensions, classification endpoints, roaming/remote servers, and other configured services create separate data paths. |
| An external subagent’s sandbox configuration is goose core’s sandbox. | ❌ FALSE | It belongs to that external agent; goose coordinates it through the configured extension/MCP path. |
| An empty `available_tools` list disables all tools from an extension. | ❌ FALSE | Empty or omitted means all tools are loaded; list names to filter availability. |
| `client_secret_key` stores the OAuth client secret directly in extension YAML. | ❌ FALSE | It names a secret resolved through environment/secret references or goose’s secret store. |
