# Enterprise Requirements & Cross-Harness Migration (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against the official `enterprise/managed-configuration`, `config-reference`, and `app-server` pages (learn.chatgpt.com).

## requirements.toml — admin-enforced policy

✅ TRUE — requirements are admin-enforced constraints that users cannot override. When a resolved value conflicts with an enforced rule, the client **falls back to a compatible value and notifies the user** (not a hard crash).
✅ TRUE — locations, confirmed verbatim: `/etc/codex/requirements.toml` on Unix (Linux **and** macOS), `%ProgramData%\OpenAI\Codex\requirements.toml` on Windows.
✅ TRUE — cloud delivery: Agent Security requirements are delivered in the cloud config bundle to managed clients — the draft's "deployed dynamically via cloud-managed administration portals" framing is accurate.
✅ TRUE — requirements hierarchy (low → high): system `requirements.toml` → Agent Security (cloud bundle) → legacy `managed_config.toml` reinterpreted fields → macOS MDM (`com.openai.codex:requirements_toml_base64`). MDM and legacy managed-device requirements rank **above** Agent Security; the device's system requirements file ranks **below** it.
✅ TRUE — requirements constrain approval policies, approvals reviewer, auto-review policy, sandbox modes / permission profiles, web search mode, managed hooks, MCP server allowlists (name **and** identity must match an approved entry, else the server is disabled), plugin marketplace sources, feature flags (`[features]` pinning), and network requirements.
✅ TRUE — `configRequirements/read` (app-server JSON-RPC) returns the live requirements state — "requirements from `requirements.toml` and/or MDM, including exact managed configuration, allowlists, pinned `featureRequirements`, and network requirements" — exactly what a client needs to disable policy-blocked UI toggles.
✅ TRUE — managed hooks: `allow_managed_hooks_only = true` (supported **only** in `requirements.toml`) skips user, project, session, and plugin hooks while allowing managed hooks; admin hooks can be defined inline under `[hooks]` in requirements with scripts delivered via `managed_dir` / `windows_managed_dir`.

## Cross-harness migration (/import)

✅ TRUE — Codex ships a migration system exposed as the `/import` slash command in the CLI and the **Settings > General > Import** flow in the desktop app; app-server clients drive it programmatically via `externalAgentConfig/detect` + `externalAgentConfig/import`.
✅ TRUE — supported sources: **Claude Code** (since v0.140.0) and **Cursor** (v0.145.0, 2026-07-21: "settings, MCP servers, plugins, sessions, commands, and project-scoped memories"; Cursor skills added in v0.147.0).
⚠️ PARTIAL — the draft lists "Claude Code, Cursor, Copilot, and Gemini CLI": only Claude Code and Cursor are documented; Copilot/Gemini CLI ingestion has no official source.
✅ TRUE — migration item types (per `externalAgentConfig/import`): config, skills, `AGENTS.md`, plugins, MCP server config, subagents, hooks, commands, and sessions; imports emit `externalAgentConfig/import/progress` and `.../completed`; plugin and session imports can complete asynchronously.
⚠️ PARTIAL — "explicitly additive … without overwriting" and "fault-isolated … flagged in a post-run telemetry report": the engine is documented as **deliberately conservative** — it skips unsupported or ambiguous artifacts rather than partially translating them (so a malformed item does not abort the batch), but neither the "additive/no-overwrite" guarantee nor a "telemetry report" appears in official docs. Treat those as ❓ unconfirmed details around a real mechanism.
✅ TRUE — `externalAgentConfig/detect` scans and returns migratable artifacts (`includeHome`, per-`cwd`); `config/batchWrite` "applies configuration edits **atomically** to the user's `config.toml` on disk" — the draft's RPC description is confirmed. (`config/value/write` writes a single key.)
