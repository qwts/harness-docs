# Corrections Log — Audit of the Source Draft

Claim-by-claim audit of "Architectural and Operational Specification of the Codex Agent Harness: CLI, Desktop App, and Configuration Protocols" (source draft). Verified 2026-10-09 against official OpenAI Codex documentation (learn.chatgpt.com / developers.openai.com / github.com/openai/codex), MCP official docs (modelcontextprotocol.io), and independent sources. Only entries with a non-✅ verdict are listed here; everything else in the draft checked out and is organized into the sibling topic files.

## ❌ False claims (do not rely on)

| # | Draft claim | Reality |
|---|---|---|
| F1 | `codex mcp add --scoped my-tool -- my-command` "forces the CLI to parse the file tree, locate the nearest Git root, and write the MCP server configuration directly to the project-local `.codex/config.toml`". | **Fabricated flag.** No `--scoped`/`--scope` option exists on `codex mcp add` (it is an open feature request, openai/codex#23487; confirmed absent in recent releases). The command always writes user-level `~/.codex/config.toml`. Project-scoped MCP servers are added by hand-editing `.codex/config.toml`, and load only for **trusted** projects. |
| F2 | `model_reasoning_effort` "Accepts 'none', 'minimal', 'low', 'medium', 'high', or 'xhigh'". | `"none"` is **not** a permitted value. Schema: `minimal \| low \| medium \| high \| xhigh` (`extra_high`/`extra-high` alias to `xhigh`); newer models additionally advertise `max` and `ultra` via `model/list`. Current Codex models coerce unsupported `minimal` to `low`. |
| F3 | "Once the server acknowledges the payload with an `initialized` notification, the session is considered active." | **Direction reversed.** The client sends `initialize`; the server responds; then the **client** emits the `initialized` notification. Requests before initialization are rejected ("Not initialized"). |

## ⚠️ Partially true / overstated

| # | Draft claim | Correction |
|---|---|---|
| P1 | `on-request` "forces the App Server to pause execution and request explicit authorization before spawning shell sub-processes, applying code patches, or executing MCP tool elicitations." | `on-request` prompts **only for actions the sandbox doesn't already allow**; sandbox-allowed commands and edits run without approval. Also omitted: `untrusted` is retired (can prevent startup) and `on-failure` is deprecated. |
| P2 | Approvals arrive as `execCommandApproval` / `applyPatchApproval` server requests answered with `"decision": "accept"/"decline"`. | Both methods are real but **deprecated legacy**. Current protocol: `item/commandExecution/requestApproval`, `item/fileChange/requestApproval`, `item/permissions/requestApproval`; current decision enums are `accept \| acceptForSession \| decline \| cancel`. |
| P3 | "If an orchestrating agent fails to intercept and respond … the agent loop will stall indefinitely, effectively deadlocking the active thread." | The turn blocks awaiting a decision, but current app-server **refuses unknown server→client requests with -32601** instead of stalling, and in non-interactive flows an action needing fresh approval **fails and surfaces the error to the parent workflow**. |
| P4 | `workspace-write` "confines the agent's write permissions strictly to the git repository root." | Writable set = **workspace root plus `/tmp` and `$TMPDIR` by default** (each excludable), expandable via `writable_roots`; not tied to the git root. |
| P5 | Model examples `gpt-5.4` (config, `-c` override, `thread/start`). | `gpt-5.4`/`gpt-5.4-mini` **retired from ChatGPT-sign-in Codex on 2026-08-31** (replacements `gpt-6-sol`/`gpt-6-luna`; current flagship example `gpt-6.1-sol`). Still usable via API-key auth. |
| P6 | `model_provider` "Accepts 'openai', 'amazon-bedrock', or 'oss'". | `model_provider` is an arbitrary **provider id from the `model_providers` table** (default `openai`). Bedrock/gateways are configured by defining provider blocks; `--oss` is a CLI flag using `oss_provider` (`lmstudio \| ollama`), not a provider value. |
| P7 | Migration ingests "Claude Code, Cursor, Copilot, and Gemini CLI"; import is "explicitly additive"; failures are "flagged in a post-run telemetry report". | Only **Claude Code** (v0.140.0+) and **Cursor** (v0.145.0+) are documented. The engine is deliberately conservative — it **skips unsupported/ambiguous artifacts** rather than aborting — but "additive/no-overwrite" and the "telemetry report" are undocumented. |
| P8 | "If a malicious actor subtly modifies the project's hook script … the App Server halts all hook execution pending a fresh security review." | Only the **changed/untrusted hook** is skipped until re-trusted (silently at hook level; a startup warning points to `/hooks`); other trusted hooks keep running. |
| P9 | Nested `AGENTS.md` files load "only when the agent's working directory matches that path", and AGENTS.md is recognized by "thousands of open-source MCP tools". | Codex loads **all** files along the root→cwd path (one per directory, override precedence, 32 KiB cap) — a deep file loads when cwd is at/below it, but there is no per-file conditional matching. AGENTS.md is a real cross-tool standard (Codex, GitHub Copilot); "MCP tools" don't recognize it. |
| P10 | "The App Server … interact[s] via … JSON-RPC 2.0 … over standard input/output (stdio) streams or WebSockets." | Also a **Unix-socket transport** (`--listen unix://`); WebSocket is **experimental and unsupported for production**. |
| P11 | "Codex explicitly requires … TOML … reserving JSON solely for auxiliary artifacts like MCP server schemas and hook definitions." | Hooks really can be JSON (`hooks.json`), but **MCP server configuration in Codex is TOML** (`[mcp_servers]`), not JSON. JSON artifacts: `hooks.json`, generated JSON Schema bundles, model catalogs. |
| P12 | Untrusted-project `.codex/` layers are "skipped silently". | Config-layer skipping is silent, but **hooks needing review print a startup warning** pointing to `/hooks`; the trust prompt itself is visible. |
| P13 | Hook payload "requires a regex `matcher`" for every hook. | `matcher` is optional for some events (e.g. `SessionEnd` examples omit it); required in practice only to target specific tools. |
| P14 | "Prompt instructions for these subagents reside in Markdown formats." | Subagent core instructions are the `developer_instructions` **TOML string**; the Markdown artifacts in the ecosystem are skills and AGENTS.md. |

## 💬 Opinion presented as architecture (kept, relabeled)

| # | Draft section | Assessment |
|---|---|---|
| O1 | Exporting/serializing config to cloud storage (`harnesses/codex/cli/`, `harnesses/codex/desktop/` in Google Drive) for multi-node reproducibility | Sound orchestration guidance; no Codex feature synchronizes configuration to cloud storage. |
| O2 | Desktop App as "visual orchestration center … highly effective for project managers overseeing multi-threaded code generation" | Editorial framing of real capabilities (subagent threads, diff review, approvals UI). |
| O3 | CLI pinning in lockfiles "guarantee[s] absolute reproducibility" | Version pinning via npm/brew is real and sensible; "absolute" is editorial. |
| O4 | Trust boundary "preventing arbitrary code-execution exploits during automated repository cloning" | Editorial framing of the real project-trust mechanism (which is exactly what the docs describe). |

## ✅ Notable claims that survived verification (surprising ones)

- The `"jsonrpc":"2.0"` header is **explicitly omitted on the wire** — sounded like a hallucination, stated verbatim in official docs.
- WebSocket rejection with error `-32001` / "Server overloaded; retry later." plus exponential backoff and jitter — confirmed verbatim.
- `personality` key with values `none \| friendly \| pragmatic` — real.
- `service_tier` accepts `fast`/`flex`; `fast` maps to API value `priority` — confirmed (CLI schema errors literally expect `fast` or `flex`).
- Hook trust recorded as `trusted_hash` under `[hooks.state]` in `~/.codex/config.toml`; `--dangerously-bypass-hook-trust` — both real, exact names.
- `/app` CLI→desktop session handoff — real (~v0.138+).
- `externalAgentConfig/detect`, `config/batchWrite` (atomic), `configRequirements/read` — all real app-server methods.
- Six-tier precedence (flags → project → profile → user → cloud-managed → system) plus the requirements override row — matches the official order.
- Project-forbidden keys incl. `openai_base_url`/`chatgpt_base_url` ignored with a startup warning — confirmed.
- `model_auto_compact_token_limit`, `developer_instructions`, `model_instructions_file`, granular `approval_policy` (with `sandbox_approval` and `mcp_elicitations` keys exactly as drafted) — all real.
- Subagent TOML trio `name`/`description`/`developer_instructions` at `.codex/agents/` and `~/.codex/agents/` — confirmed.
- MCP `enabled_tools`/`disabled_tools` allow/deny lists — confirmed (`disabled_tools` applied after `enabled_tools`).
- `gpt-6.1-sol` — a real, current model name (the draft's other example was retired, but this one is live).

## Source-doc citation quality

The draft's footnotes are mixed: [2], [3], and [5] are the **same swequiz.com article cited three times** (padding); [4] (official app-server doc) is the strongest source and correctly used for the protocol section; [6] (a developer gist on the JSON-RPC interface) is solid for protocol/hooks/subagents coverage; [1] and [7] are blog roundups. Notably, the enterprise/hooks/migration claims rest on the gist and blogs rather than official docs — yet almost all of them turned out true when checked. The hard errors (F1 `--scoped` flag, F2 `"none"` effort value, the Copilot/Gemini import claim) appear in **none** of the cited sources. Conclusion: **architecture, protocol, and configuration-topology material is trustworthy; command flags, enum lists, and method names required correction.**
