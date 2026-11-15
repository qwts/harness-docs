# Configuration, providers, and models

**Verified:** 2026-11-15

## The Hermes home

Everything lives under `~/.hermes/` (`$HERMES_HOME` overrides):

| Path | Contents |
|---|---|
| `config.yaml` | Main settings (YAML) |
| `.env` | Credentials and UPPER_SNAKE settings |
| `auth.json` | OAuth/session credentials |
| `SOUL.md` | Agent identity (loaded only from here) |
| `memories/` | `MEMORY.md` + `USER.md` |
| `skills/` | Skill catalog (plus `.hub/`, `.bundled_manifest`) |
| `cron/` | `jobs.json`, `output/`, `executions.db`, `scripts/` under home |
| `state.db` | SQLite session store (WAL, FTS5) |
| `sessions/`, `logs/`, `pending/`, `mcp-tokens/`, `checkpoints/` | Transcripts, logs, staged writes, OAuth tokens, shadow git store |
| `profiles/<name>/` | Named profiles — a full home each (`hermes -p <name>`) |

Precedence for a setting: **CLI args > `config.yaml` > `.env` > built-in defaults**. The docs phrase the config CLI as a router: `hermes config set` sends UPPER_SNAKE keys to `.env` and dotted keys to `config.yaml`.

## Managed scope

Admins pin a baseline in `/etc/hermes/` (`HERMES_MANAGED_DIR` relocates; Hermes ignores that variable if set in `.env`/config, logging a warning): `config.yaml` for the managed config layer, `.env` for the managed env layer, with “directory mode 0755, files 0644” — “readable by everyone, writable only by an administrator. That filesystem permission is the enforcement mechanism.”

- Managed values win “for exactly the keys it pins. Everything else stays fully user-controlled” — merging is leaf-level (pinning `model.default` does not freeze `model.fallback`).
- Managed scope “deliberately wins over the shell environment too.”
- Changing a pinned key is refused: “Cannot set ‘model.default’: it is managed by your administrator.”
- “A malformed managed file is logged loudly and ignored — it never blocks startup.”
- v1 limits, quoted: “Enforcement is filesystem permissions only”; the managed `.env` is world-readable, so “use it for shared, non-sensitive values … rather than high-sensitivity secrets”; agent tools are not hard-blocked from managed values — “a management-convenience boundary against a normal user, not an un-escapable sandbox.”

## Secrets

The bootstrap token for a secret manager lives in `.env`; other provider keys can move to external sources:

- Bundled: Bitwarden Secrets Manager (`bws` CLI) and 1Password (`op://` references), plus a command helper for “any CLI vault (keepassxc-cli, secret-tool, pass, custom scripts)” printing `KEY=VALUE` lines; every injected credential is “labelled with its origin.”
- Precedence: “Your .env / shell wins by default”; a source replaces an existing value only with `override_existing: true` (Bitwarden defaults to true); “first source wins” on conflicts, with a startup warning; `secrets.preserve_existing` pins specific vars; profile aliasing hydrates `FOO_<PROFILE>` into `FOO` for credential-shaped suffixes only (`*_API_KEY`, `*_TOKEN`, `*_SECRET`, `*_KEY`, `*_PASSWORD`).
- Child processes are sanitized: terminal commands, `execute_code` sandboxes, and no-agent cron scripts see only `terminal.env_passthrough` names (or a skill’s `required_environment_variables`) — “a profile’s declared value never reaches another profile’s children”, and Hermes-managed provider credentials cannot be declared.

## Providers and models

The provider registry: openrouter, nous, openai-codex, anthropic, gemini, qwen-oauth, zai, kimi-coding, minimax-oauth, deepseek, xai-oauth, ollama-cloud, bedrock, azure-foundry, plus user-defined `custom providers` entries — each slot takes `provider`/`model`/`base_url`/key in the same universal shape. `auto` in an auxiliary slot means the main model.

**Nous Portal** (`hermes setup --portal`): Nous’s hosted access (300+ models), the “lowest-friction option for unattended runs since OAuth refresh is automatic.” Paid subscribers additionally get the **Tool Gateway** — “web search, image generation, TTS, and browser automation … no separate API keys needed.”

**Auxiliary model slots** (each independently routable): `vision`, `title_generation`, `compression`, `approval`, `mcp`, `tts_audio_tags`, `voice_chat`, `triage_specifier`, `kanban_decomposer`, `profile_describer`, `curator`, `delegation`, `background_review`, `review`, `group_router`. The background review runs on the main model by default, replaying a conversation that is “already warm in the prompt cache”; pointing it at a different model switches it to a compact conversation digest (~3–5× cheaper in benchmarks), and honors `reasoning_effort` only on that different-model route — a same-model review “always inherits the parent’s reasoning effort.” Its input budget: 75% of the review model’s context window, capped at 600,000 tokens, falling back to 120,000 when the window cannot be resolved.

**Model aliases** — two formats:

```yaml
model_aliases:
  fav:
    model: claude-sonnet-4.6
    provider: anthropic
  ollama-qwen:
    model: qwen3-coder:30b
    provider: custom
    base_url: http://localhost:11434/v1
    key_env: THETA_API_KEY        # or api_key: "${THETA_API_KEY}"
```

or `hermes config set model.aliases.fav anthropic/claude-opus-4.6`. User aliases shadow built-in short names; `/model <alias>` switches session-only by default (`--global` persists; `model.persist_switch_by_default: true` flips the default).

**Fallbacks:** `fallback_providers` (legacy `fallback_model`) rescues unpinned jobs and sessions; anything pinned — a cron job with its own provider/model, a delegated child with a pinned route — “never falls back to the global chain.”

## Environment variables

| Variable | Role |
|---|---|
| `HERMES_HOME` | Relocates the entire home |
| `HERMES_MANAGED_DIR` | Relocates managed scope (ignored from `.env`/config by design) |
| `HERMES_YOLO_MODE` | YOLO approval bypass (same as `--yolo`) |
| `HERMES_SAFE_MODE` | Skips plugins, shell hooks, outbound webhooks; never gateway hooks |
| `HERMES_IGNORE_RULES` | Skips AGENTS.md/SOUL.md/.cursorrules/memory/auto-load skills |
| `HERMES_ACCEPT_HOOKS` | Approves shell hooks without the first-use prompt |
| `HERMES_CRON_SCRIPT_TIMEOUT` / `HERMES_CRON_MEDIA_SEND_TIMEOUT` | Cron script (default 3600 s) and per-attachment (default 300 s) timeouts |
| `SUDO_PASSWORD` | Non-interactive sudo on messaging/headless surfaces |
| `TERMINAL_SSH_HOST` / `TERMINAL_SSH_USER` / `TERMINAL_SSH_KEY` | SSH terminal backend |
| `GITHUB_TOKEN` | Raises the Skills Hub GitHub API limit (60→5,000 req/h) |
| `HERMES_PEER_<NAME>_KEY` | Peer gateway credential (`hermes peer add`) |
| `VERCEL_TOKEN` / `VERCEL_PROJECT_ID` / `VERCEL_TEAM_ID` | Vercel Sandbox backend (OIDC token for one-off use) |
| Provider keys (`OPENAI_API_KEY`, `TELEGRAM_BOT_TOKEN`, `DISCORD_HOME_CHANNEL`, …) | `.env`-scoped credentials and routing targets |

## Official sources

- [Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration) · [Configuring Models](https://hermes-agent.nousresearch.com/docs/user-guide/configuring-models) · [Managed Scope](https://hermes-agent.nousresearch.com/docs/user-guide/managed-scope) · [Secrets](https://hermes-agent.nousresearch.com/docs/user-guide/secrets) · [Profiles](https://hermes-agent.nousresearch.com/docs/user-guide/profiles)
