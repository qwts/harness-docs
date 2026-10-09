# Core config.toml Schema (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against the official Configuration Reference, Models, and Agent approvals & security pages (learn.chatgpt.com).

## Model & inference keys (draft table, corrected)

| Key | Draft claim | Verdict & correction |
|---|---|---|
| `model` | e.g. "gpt-6.1-sol", "gpt-5.4" | ⚠️ PARTIAL — the key is real; `gpt-6.1-sol` is the current flagship example. `gpt-5.4`/`gpt-5.4-mini` **retired from ChatGPT-sign-in Codex on 2026-08-31** (replacements: `gpt-6-sol`, `gpt-6-luna`); still usable via API key |
| `model_provider` | accepts "openai", "amazon-bedrock", or "oss" | ⚠️ PARTIAL — the value is a provider id from your `model_providers` table (default `openai`). Bedrock/gateways are configured by defining provider blocks; `--oss` is a CLI flag whose backend is set by `oss_provider` (`lmstudio | ollama`), not a `model_provider` value |
| `model_reasoning_effort` | accepts none/minimal/low/medium/high/xhigh | ⚠️ PARTIAL — `"none"` is **not** a value (❌); schema: `minimal | low | medium | high | xhigh` (`extra_high`/`extra-high` alias to `xhigh`); newer models additionally advertise `max` and `ultra` (discover via `model/list`); current models coerce unsupported `minimal` to `low` |
| `model_auto_compact_token_limit` | threshold triggering automatic history compaction (e.g., 200000) | ✅ TRUE — real key; unset uses model defaults. Related: `model_auto_compact_token_limit_scope` (`total` vs `body_after_prefix`) |
| `personality` | accepts "none", "friendly", "pragmatic" | ✅ TRUE — exact values confirmed; applies to models advertising `supportsPersonality`; overridable per thread/turn and via `/personality` |
| `service_tier` | accepts "fast" or "flex" | ✅ TRUE — `fast` maps to API value `priority`; other tiers as advertised by the active model (CLI schema errors literally expect `fast` or `flex`) |

Also real (not in draft): `model_context_window`, `review_model`, `model_catalog_json`, `oss_provider`, `compact_prompt`.

## Custom providers

✅ TRUE — an enterprise LLM proxy / data-residency gateway / Bedrock endpoint is defined as a `model_providers.<id>` block (with `base_url`, `env_key`, bearer-token command, etc.) rather than by manipulating built-in routing keys. This is how the App Server formats auth headers for the remote endpoint.

## Sandbox

| Draft claim | Verdict |
|---|---|
| `sandbox_mode` = `read-only` / `workspace-write` / `danger-full-access` | ✅ TRUE |
| `read-only` restricts to inspecting files and non-mutating commands; no writes | ✅ TRUE |
| `workspace-write` confined "strictly to the git repository root" | ⚠️ PARTIAL — the writable set is the **workspace root plus `/tmp` and `$TMPDIR` by default** (each excludable via `exclude_slash_tmp` / `exclude_tmpdir_env_var`); it is not tied to the git root |
| `sandbox_workspace_write.writable_roots` array expands the boundary | ✅ TRUE |
| `danger-full-access` disables isolation entirely (unrestricted read/write) | ✅ TRUE |
| Outbound network is blocked by default; requires `sandbox_workspace_write.network_access = true` | ✅ TRUE |

## Approval policy

| Draft claim | Verdict |
|---|---|
| `on-request` pauses before spawning shell sub-processes, applying code patches, or executing MCP elicitations | ⚠️ PARTIAL — `on-request` prompts **only for actions the sandbox doesn't already allow**; sandbox-allowed commands and edits run without approval. MCP elicitation prompts surface under granular `mcp_elicitations = true` |
| `never` grants autonomy without prompting | ✅ TRUE — intended for non-interactive runs |
| (value list) | ⚠️ PARTIAL — the draft omitted that `untrusted` is **retired** (can prevent startup) and `on-failure` is **deprecated**; supported values are `on-request` and `never`, plus the granular table |
| Granular override, e.g. `approval_policy = { granular = { sandbox_approval = false, mcp_elicitations = true } }` | ✅ TRUE — real schema. Full granular keys: `sandbox_approval`, `rules`, `mcp_elicitations`, `request_permissions`, `skill_approval` (true = prompts in that category may surface; false = auto-rejected) |

Related real keys (not in draft): `approvals_reviewer` (`user | auto_review`), `auto_review.policy` / `auto_review.extra_policy`, `allow_login_shell`.

## Instruction keys

| Key | Verdict |
|---|---|
| `developer_instructions` — raw string injected into the session | ✅ TRUE ("Additional developer instructions injected into the session") |
| `model_instructions_file` — path to a Markdown file of behavioral guidelines | ✅ TRUE ("Replacement for built-in instructions instead of `AGENTS.md`") |
| `model_instructions_file` "supersedes legacy keys like `experimental_instructions_file`" | ❓ UNVERIFIED — no current official doc mentions `experimental_instructions_file`; the historical key existed in older releases, but the supersession relationship has no official source |
| `instructions` | reserved for future use (prefer `model_instructions_file` or `AGENTS.md`) |
