# Corrections — common misinformation about Kiro

No upstream draft existed for this harness; this file audits claims commonly repeated about Kiro (many from stale Amazon Q-era material), checked 2026-10-09 against kiro.dev/docs.

## FALSE

| ID | Claim | Reality |
|---|---|---|
| F1 | "Kiro is a VS Code extension / just the IDE." | Kiro is a unified agent harness with five surfaces (IDE, CLI, Web, Mobile, Crew); the IDE is a Code OSS desktop app, not an extension, and `.kiro/` config is shared by all surfaces. |
| F2 | "Kiro hooks are defined in agent config / chat settings." | Pre-1.0/3.0 formats were embedded; current format is standalone `.kiro/hooks/*.json` files with PascalCase triggers. `kiro-cli agent migrate` converts CLI 2.x. |
| F3 | "MCP servers go in one global mcp.json." | Three scopes: workspace MCP config, user MCP config, and per-agent `mcpServers` (independent unless `includeMcpJson: true`). |
| F4 | "`kiro://` install links add MCP servers silently." | They open a confirmation dialog showing command + env/header names (values hidden); nothing is written until you confirm. |
| F5 | "Kiro is Amazon Q Developer renamed and identical." | Kiro descends from Q (migration docs exist) but the harness, `.kiro/` layout, hooks format, and steering/inclusion system are Kiro-specific and versioned. |

## PARTIAL

| ID | Claim | Reality |
|---|---|---|
| P1 | "Steering files and AGENTS.md are the same thing." | Both feed context, but AGENTS.md is always-included with no frontmatter modes; steering `.md` supports `always`/`fileMatch`/`manual`/`auto`. They coexist. |
| P2 | "Steering inclusion modes work everywhere." | CLI V3 supports all four; CLI V1/V2 only auto-load `always` files. |
| P3 | "Global steering applies in cloud sessions." | `~/.kiro/steering/` is IDE/CLI only — the Web sandbox can't read it; upload via Configuration Sync. |
| P4 | "The matcher in a hook matches anything." | It's a regex matched against the tool name (tool triggers) or file path (file triggers); omit to match all. |
| P5 | "Hooks just run shell commands." | `action.type` is `command` or `agent` — agent actions inject a prompt into the conversation instead of spawning a shell. |
| P6 | "MCP tool names are freeform." | Validation: ≤64 chars incl. prefix, `^[a-zA-Z][a-zA-Z0-9_]*$`, non-empty description; failing tools are excluded entirely. >10k-char descriptions warn. |
| P7 | "MCP is available on all surfaces." | Local+remote MCP works on IDE/CLI/Web — not Mobile. Hooks likewise skip Mobile. |
| P8 | "Commands receive args as argv." | Command hooks receive session context as **JSON on STDIN**, not argv. |
| P9 | "Every trigger supports confirmation prompts." | `confirm`/`confirmCommand` is documented for `Stop` command hooks. |
| P10 | "All surfaces share identical config." | `.kiro/` is shared, but permissions and primary-agent selection are explicitly surface-specific. |

## UNVERIFIED

| ID | Claim |
|---|---|
| U1 | Exact `.kiroignore` file format (feature documented; syntax not verified). |
| U2 | Full sub-agent / powers / skills schema (listed capabilities only). |
| U3 | Whether desktop-class Mobile limits extend to specs execution (docs show steering/MCP/hooks only). |
