# AGENTS.md, Subagents, MCP & Lifecycle Hooks (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against the official `agent-configuration/agents-md`, `agent-configuration/subagents`, `extend/mcp`, and `hooks` pages (learn.chatgpt.com).

## AGENTS.md

✅ TRUE — Codex builds an instruction chain at session start ("reads AGENTS.md files before doing any work"): global `~/.codex/AGENTS.md` (or `AGENTS.override.md`), then project files from the repo root **down to the cwd** (at most one file per directory: `AGENTS.override.md` > `AGENTS.md` > `project_doc_fallback_filenames`), concatenated root→cwd with a 32 KiB default cap (`project_doc_max_bytes`).
⚠️ PARTIAL — draft: nested `AGENTS.md` files load "only when the agent's working directory matches that path." Corrected: Codex loads **every** file along the root→cwd path (so a deep file loads only when cwd is at or below it — the draft's effect is right), but there is no per-file conditional matching; all files from root to cwd are merged into the prompt, bounded by the byte cap. Files closer to cwd appear later and override earlier guidance.
⚠️ PARTIAL — "recognized natively by Codex, GitHub Copilot, and thousands of open-source MCP tools": `AGENTS.md` is a real cross-tool standard (agents.md) adopted by Codex and GitHub Copilot; "MCP tools" do not recognize AGENTS.md — overstated.

## Custom subagents

✅ TRUE — multi-agent delegation is native: the primary agent spawns specialized subagents in parallel **agent threads** (built-ins: `default`, `worker`, `explorer`; `/agent` inspects/switches threads; the parent collects results; subagents inherit the parent's sandbox policy and live runtime overrides).
✅ TRUE — declarations are standalone **TOML** files: `~/.codex/agents/*.toml` (personal) or `.codex/agents/*.toml` (project-scoped). Codex loads them as configuration layers for spawned sessions.
✅ TRUE — the draft's field table matches the official schema:

| Field | Type | Required | Reality |
|---|---|---|---|
| `name` | string | Yes | identifier used when spawning (filename matching is convention only; the field is the source of truth) |
| `description` | string | Yes | human-facing guidance for when Codex should use this agent (steers the parent's delegation decision) |
| `developer_instructions` | string | Yes | core instructions that define the agent's behavior |

Optional: any supported config.toml keys — `model`, `model_reasoning_effort`, `sandbox_mode`, `mcp_servers`, `skills.config`. Defaults live under the `[agents]` table (`max_concurrent_threads_per_session`, `default_subagent_model`, …).
⚠️ minor — draft: "prompt instructions for these subagents reside in Markdown formats" — `developer_instructions` is a TOML string (skills are the Markdown artifact); imprecise.

## MCP

✅ TRUE — MCP servers are declared as `[mcp_servers.<name>]` TOML blocks in `config.toml` (user-level, or project-scoped `.codex/config.toml` for trusted projects) — Codex deviates from the JSON `.mcp.json` convention used by several other harnesses.
✅ TRUE — `codex mcp add <name> [--env K=V ...] -- <command>` scaffolds server config; also `codex mcp list`, `codex mcp login <name>` (OAuth), `/mcp` in the TUI. HTTP servers: `codex mcp add <name> --url https://... --oauth-client-id ...`.
❌ FALSE — the draft's `codex mcp add --scoped my-tool -- my-command` ("parses the file tree, locates the nearest Git root, writes to project-local `.codex/config.toml`"): **no `--scoped`/`--scope` flag exists** (open feature request openai/codex#23487; confirmed absent in recent releases). `codex mcp add` always writes user-level `~/.codex/config.toml`. Project-scoped servers require hand-editing `.codex/config.toml` — and load only for trusted projects, so "CI pipelines cloning the repository instantly inherit identical tooling" is gated by the trust prompt.
✅ TRUE — per-server tool allow/block lists: `enabled_tools` (allow list) and `disabled_tools` (deny list, applied **after** `enabled_tools`).
Real per-server keys the draft missed: `startup_timeout_sec` (default 10), `tool_timeout_sec` (default 60), `enabled`, `required`, `default_tools_approval_mode`, `tools.<tool>.approval_mode`, bearer/OAuth options, `env_vars`.
✅ TRUE — transports: stdio (local command) and streamable HTTP, with bearer/OAuth (incl. CIMD/DCR) and ChatGPT session auth.

## Lifecycle hooks

✅ TRUE — hooks run scripts or MCP tools during the agent loop; they are defined in `hooks.json` or inline `[hooks]` tables in `config.toml`, discovered next to **every active config layer** (`~/.codex/hooks.json`, `<repo>/.codex/hooks.json`, `~/.codex/config.toml`, `<repo>/.codex/config.toml`). Multiple matching hooks all run; hooks from different layers are merged, not replaced.
✅ TRUE — hook events include the draft's five — `SessionStart`, `PreToolUse`, `PostToolUse`, `PermissionRequest`, `SessionEnd` — plus `SubagentStart`/`SubagentStop`, `PreCompact`/`PostCompact`, `UserPromptSubmit`, `Stop`, `Interrupt`.
✅ TRUE — handler shape: a regex `matcher` group plus handlers declaring `type` = `command` or `mcp_tool` with the executable command (MCP hooks add `server` + `tool`). The draft's `PostToolUse` hook matching `^apply_patch$` to run a linter after writes is valid usage (`apply_patch` is a canonical hook tool name).
✅ TRUE — hook trust: non-managed hooks must be reviewed and trusted before running. Trust is recorded against the hook's **current hash** as `trusted_hash` under `[hooks.state]` in `~/.codex/config.toml`; review/approve via `/hooks` (CLI) or the review flow; `--dangerously-bypass-hook-trust` runs enabled hooks without persisted trust for one invocation (exact flag name confirmed).
⚠️ PARTIAL — "if a malicious actor modifies the hook script, the App Server halts ALL hook execution pending a fresh security review": only the **changed/untrusted hook** is skipped until re-trusted; other trusted hooks keep running. A startup warning points to `/hooks` when hooks need review.
⚠️ nuance — hooks are enabled by default (`features.hooks`; `codex_hooks` is a deprecated alias). Managed hooks from requirements.toml/MDM/cloud are trusted by policy and cannot be disabled from the user hook browser; project-local hooks load only for trusted projects.
