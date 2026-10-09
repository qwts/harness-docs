# Configuration Locations & Precedence (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Claims below inherit from the source draft unless noted. Verified 2026-10-09 against the official `config-basic`, `config-advanced`, `config-reference`, and `enterprise/managed-configuration` pages (learn.chatgpt.com).

## Where configuration lives

| Layer | Location | Verdict |
|---|---|---|
| User config | `~/.codex/config.toml` (`$CODEX_HOME`) | ✅ TRUE — user-level default; the CLI, IDE extension, and desktop app share the same configuration layers |
| Project config | `.codex/config.toml` (repo root down to subfolders) | ✅ TRUE — repository-scoped overrides; **trusted projects only** |
| Profiles | `$CODEX_HOME/profile-name.config.toml`, selected via `--profile profile-name` | ✅ TRUE — profile names may contain letters, numbers, hyphens, underscores; use top-level keys (do not nest under `[profiles.name]`) |
| Cloud-managed defaults | delivered for the signed-in workspace | ✅ TRUE — these are *defaults*, not enforced policy; users can still override them |
| System config | `/etc/codex/config.toml` (Unix) | ✅ TRUE |
| Enterprise requirements | `requirements.toml` (system / MDM / Agent Security) | ✅ TRUE — constraints, not defaults; see [enterprise-and-migration.md](enterprise-and-migration.md) |

## Load order

✅ TRUE — the draft's six-tier precedence table matches the official order. The draft omitted the final built-in defaults tier:

1. CLI flags and `--config` (`-c key=value`) overrides — ✅ bypass all file-based settings
2. Project `.codex/config.toml` — ✅ ordered from the project root **down to the current working directory; the closest file wins** (the draft's "walks backward from cwd; closest to execution directory wins" states the same semantics)
3. Profile file selected with `--profile`
4. User `~/.codex/config.toml`
5. Cloud-managed `config.toml` defaults (signed-in workspace)
6. System config `/etc/codex/config.toml` (Unix)
7. Built-in defaults (not in the draft)

✅ TRUE — a key defined in multiple layers resolves to the highest-priority layer.

## Trust boundary

✅ TRUE — Codex loads project-scoped `.codex/` layers **only when the project is trusted**. In an untrusted project, project config, project hooks, project MCP declarations, and custom agents are skipped entirely — a repository-embedded `.codex/config.toml` cannot disable sandbox constraints or exfiltrate environment variables via config.
✅ TRUE — a per-project entry `trust_level = "untrusted"` in **user-level** `~/.codex/config.toml` disables project-local configuration and enforces per-command approvals (this replaces the retired `approval_policy = "untrusted"`).
⚠️ PARTIAL — "skipped silently": the config-layer skip itself is silent, but hooks that need review print a startup warning pointing to `/hooks`.

## Keys forbidden in project-local config.toml

✅ TRUE (confirmed verbatim) — Codex ignores these keys when they appear in a project `.codex/config.toml` and prints a startup warning for each: `openai_base_url`, `chatgpt_base_url`, `apps_mcp_product_sku`, `model_provider`, `model_providers`, `notify`, `profile`, `profiles`, `experimental_realtime_ws_base_url`, `otel`. Provider, notification, and telemetry routing keys belong in user-level config.
✅ TRUE — the draft's security rationale matches the documented behavior: a project repo cannot repoint authentication/telemetry routing (prevents credential exfiltration to unauthorized endpoints).

## Format: TOML vs JSON

✅ TRUE — harness-level configuration is TOML (`config.toml`; custom agent declarations are TOML too).
⚠️ PARTIAL — "reserving JSON solely for auxiliary artifacts like MCP server schemas and hook definitions": hooks really can be JSON (`hooks.json`), but MCP server configuration in Codex is TOML (`[mcp_servers]` tables), not JSON. Actual JSON artifacts: `hooks.json`, generated app-server JSON Schema bundles, optional JSON model catalogs.

💬 OPINION — the draft's recommendation to export/serialize configuration to centralized cloud storage (e.g., a `harnesses/codex/cli/` tree in Google Drive) is orchestration guidance, not harness behavior. Treating config as a deployable artifact is sound practice; no Codex feature synchronizes configuration to cloud storage.
