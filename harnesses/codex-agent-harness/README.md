# Codex Agent Harness — CLI, Desktop App & App Server (Verified)

> **Status:** Reorganized + fact-checked 2026-10-09 against official OpenAI Codex documentation.
> **Provenance:** Derived from a source draft ("Architectural and Operational Specification of the Codex Agent Harness: CLI, Desktop App, and Configuration Protocols"), which was audited claim-by-claim. False and unsupported claims were corrected or removed; surviving claims are marked with a verdict.
> **Intended consumer:** Autonomous agents, deployment orchestrators, and programmatic clients configuring or embedding OpenAI Codex (CLI, ChatGPT desktop app, app-server JSON-RPC).

## Verification verdict legend

| Verdict | Meaning |
|---|---|
| ✅ TRUE | Confirmed against official docs (learn.chatgpt.com / developers.openai.com / github.com/openai/codex / modelcontextprotocol.io) or multiple independent sources. |
| ⚠️ PARTIAL | Contains truth but is imprecise, overstated, or missing critical nuance. Corrected version given. |
| ❌ FALSE | Refuted by official docs or multiple independent sources. Do not rely on. |
| 💬 OPINION | Editorial/blog commentary, not harness-specified behavior. Safe as guidance, not as spec. |
| ❓ UNVERIFIED | Plausible but no official source found. Treat as unconfirmed. |

## File map

| File | Contents |
|---|---|
| [config-and-precedence.md](config-and-precedence.md) | config.toml locations, 6-layer load order + requirements override, profiles, project trust boundary, project-forbidden keys |
| [config-schema.md](config-schema.md) | Core config.toml schema: model/provider keys, reasoning effort, personality, service_tier, sandbox modes, approval policies (incl. granular), instruction keys |
| [runtime-app-server.md](runtime-app-server.md) | App Server (Rust, JSON-RPC 2.0), Thread/Turn/Item, transports, approval interception, schema generation, CLI vs desktop surfaces |
| [agents-mcp-hooks.md](agents-mcp-hooks.md) | AGENTS.md discovery, custom subagents (TOML), MCP servers ([mcp_servers], codex mcp commands, tool allow/deny lists), lifecycle hooks + hash trust |
| [enterprise-and-migration.md](enterprise-and-migration.md) | requirements.toml / MDM / Agent Security, configRequirements/read, cross-harness /import and externalAgentConfig RPC methods |
| [corrections.md](corrections.md) | Full audit log: every false/overstated claim from the source draft, with the correction |

## Top corrections (read first if skimming)

The source draft was **unusually accurate on architecture and configuration topology** — the precedence table, requirements.toml paths, hooks trust model, and app-server protocol mostly checked out. Defects found:

1. **❌ `codex mcp add --scoped my-tool -- my-command` does not exist.** No `--scoped`/`--scope` flag ships on `codex mcp add` (open feature request openai/codex#23487). The command always writes user-level `~/.codex/config.toml`; project-scoped MCP servers must be hand-added to `.codex/config.toml` (trusted projects only).
2. **❌ `model_reasoning_effort` does not accept `"none"`.** Schema values: `minimal | low | medium | high | xhigh` (`extra_high` aliases to `xhigh`); newer models additionally advertise `max` and `ultra` via `model/list`. Current models coerce unsupported `minimal` to `low`.
3. **❌ The `initialized` handshake direction is reversed.** The client sends `initialize`, the server responds, then the **client** emits the `initialized` notification.
4. **⚠️ `on-request` approval semantics are overstated.** Codex does not pause before every command/patch; sandbox-allowed actions run without approval — only actions outside the sandbox's permissions prompt. (`untrusted` is retired; `on-failure` is deprecated.)
5. **⚠️ `execCommandApproval` / `applyPatchApproval` are deprecated legacy methods.** Current app-server versions use `item/commandExecution/requestApproval`, `item/fileChange/requestApproval`, and `item/permissions/requestApproval`.
6. **⚠️ `/import` covers Claude Code and Cursor — not Copilot or Gemini CLI.** Copilot/Gemini ingestion and the "post-run telemetry report" have no official source.
7. **⚠️ `gpt-5.4` is retired** from ChatGPT-sign-in Codex (2026-08-31; replaced by `gpt-6-sol`/`gpt-6-luna`). `gpt-6.1-sol` is the current flagship example in official docs.
8. **⚠️ `workspace-write` is not "strictly confined to the git repository root."** Writable set = workspace root plus `/tmp` and `$TMPDIR` by default (both excludable), expandable via `sandbox_workspace_write.writable_roots`.
9. **⚠️ A modified hook does not "halt all hook execution."** Only the changed/untrusted hook is skipped until re-trusted; other hooks keep running. (The `trusted_hash` / `[hooks.state]` storage, `/hooks` review flow, and `--dangerously-bypass-hook-trust` flag are all real — surprisingly, exactly as drafted.)
10. **💬 The Google-Drive / `harnesses/codex/` repo-mapping orchestration narrative is opinion** — sound deployment guidance, not harness-specified behavior.

## Quick-reference: where to write configuration

| Target | Path |
|---|---|
| User config (all local clients share it) | `~/.codex/config.toml` (`$CODEX_HOME`) |
| Named profile layer | `$CODEX_HOME/profile-name.config.toml`, selected via `--profile` |
| Project overrides | `.codex/config.toml` — trusted projects only; project root → cwd, closest wins |
| Project-forbidden keys (ignored + startup warning) | `openai_base_url`, `chatgpt_base_url`, `model_provider(s)`, `notify`, `profile(s)`, `otel`, … |
| MCP servers | `[mcp_servers.<name>]` in user or project `config.toml`; `codex mcp add` writes user config only |
| Lifecycle hooks | `hooks.json` or inline `[hooks]` next to any active config layer (`~/.codex/hooks.json`, `<repo>/.codex/hooks.json`, …) |
| Custom subagents | `~/.codex/agents/*.toml` (personal), `.codex/agents/*.toml` (project-scoped) |
| Global agent instructions | `~/.codex/AGENTS.md` (or `AGENTS.override.md`) |
| Project agent instructions | `AGENTS.md` at repo root + nested files along the path to cwd |
| Enterprise requirements | `/etc/codex/requirements.toml` (Unix/macOS), `%ProgramData%\OpenAI\Codex\requirements.toml` (Windows); macOS MDM: `com.openai.codex:requirements_toml_base64` |
| System config defaults | `/etc/codex/config.toml` (Unix) |
| Hook trust records | `trusted_hash` entries under `[hooks.state]` in `~/.codex/config.toml` |

## Sources consulted (verification, 2026-10-09)

- Official Codex docs (learn.chatgpt.com / developers.openai.com): `config-basic`, `config-advanced`, `config-reference`, `app-server`, `hooks`, `agent-approvals-security`, `agent-configuration/agents-md`, `agent-configuration/subagents`, `extend/mcp`, `enterprise/managed-configuration`, `models`, `codex/cli`, `cli-customization`, `open-source`
- `github.com/openai/codex` — Apache-2.0 LICENSE; issues #23487, #15099, #26604, #27297, #31709, #47283
- MCP official docs: `modelcontextprotocol.io`
- Independent cross-checks: codex.danielvaughan.com, promptfoo.dev docs, dev.to, zenn.dev, swequiz.com, blakecrosley.com, Unwait, CodexZH, third-party integration repos (chimaera, solenta, nightcore, codex-wrapper)
