# Configuration, settings, and providers

**Verified:** 2026-11-15

## The `~/.commandcode/` layout

Configuration is JSON under `~/.commandcode/` (home resolved from `HOME`/`USERPROFILE`) — “Files appear the first time something is written,” so a fresh install creates none:

| File | Purpose |
|---|---|
| `config.json` | Preferences: `provider`, `model`, `theme`, `compactMode`, `reasoningEffort`, `featureModels`, `tasteLearning`, `artifactsEnabled`, machine-written `installed`/`firstMessageSent` |
| `settings.json` | “hooks, permissions, MCP” plus `model`, `providers`, `skills`, `mods`, `disabledSkills`, `subagentMaxTurns`, `attribution`, `disableSkillShellExecution`, `disableScratchpad` |
| `auth.json` | Credentials, “written 0600”, “don’t edit”; never sent to third parties |
| `providers.json` | Custom BYOK providers |
| `keybindings.json` | Key overrides; `/reload` applies mid-session |
| `mcp.json` | User-scope MCP servers |
| `cron/jobs.json` | Scheduled jobs (“managed”) |
| `projects/{project}/config.json` | Machine-written per-project state (taste onboarding flags) |
| `projects/{project}/mcp.json` | Local MCP servers |

Project-level: `.commandcode/settings.json` (“commit this”), `.commandcode/settings.local.json` (“gitignore this”), and `.mcp.json` at the project root (“commit this”).

## Precedence

Settings resolve high → low: project `settings.local.json`, project `settings.json`, user `settings.json`, then `config.json` — which contributes only legacy fields (`model`, `theme`, `compactMode`, `tasteLearning`, `featureModels`, `reasoningEffort`). “Maps are deep-merged and scalars overwrite” — except permission rule lists, which **union** across every scope rather than being overwritten. Model resolution is its own chain: launch `--model` flag → explicit `/model` for the session → last `/model` default → built-in default, with a running session reading its default from `config.json` once at startup.

## Environment variables

`COMMAND_CODE_API_KEY` (provider key; “overrides auth.json when set”), `CMD_ZDR=1` (“enforces zero data retention on every request”), `CMD_LOCAL_ONLY=1`, `CMD_DISABLE_REMOTE_MODEL_CATALOG=1`, `CMD_DISABLE_NOTICES=1`, `COMMANDCODE_SKIP_UPDATES`, `COMMANDCODE_SCRATCHPAD`/`COMMANDCODE_SCRATCHPAD_BASE`, `MCP_TOOL_TIMEOUT` (ms), `MAX_MCP_OUTPUT_TOKENS` (default 25000), `DO_NOT_TRACK`, and `HOME`/`USERPROFILE`.

## BYOK providers

BYOK is beta: “Any OpenAI or Anthropic compactible API can serve models in Command Code” (the docs’ spelling), and for custom providers “requests go straight from your machine to that provider’s endpoint with your key, never through Command Code’s servers.” Providers are managed with `/connect` — “supports 150+ providers out of the box” with prefilled endpoint/wire — or by editing `~/.commandcode/providers.json`. Three wire formats: `openai-completions` (`/chat/completions`), `openai-responses` (`/responses`), and `anthropic-messages` (`/v1/messages`).

Provider entries need `baseURL` and `models`; `apiKey` accepts a `"$ENV_VAR"`, `"{env:VAR}"`, `"!command"` reference, or `false` for keyless local servers (Ollama’s `http://localhost:11434/v1` is the documented example). Raw secrets in the file “is refused with a warning” — keys pasted in `/connect` land in `auth.json` (0600), and a stored key wins over the file reference per request. Malformed entries are skipped with a warning and never break siblings; unparseable JSON makes providers vanish for that run and writers refuse to touch the file. Project/repo-level providers are not supported: “BYOK providers are set only on the user-level.” Signing in also works for OpenAI ChatGPT/Codex (`cmd login openai`) and GitHub Copilot (`cmd login copilot`).

## Attribution and model routing

Commits carry a default `Co-authored-by: CommandCodeBot <noreply@commandcode.ai>` trailer; the `attribution` setting takes a custom single-line `Co-authored-by:` trailer (max 200 chars) or `""` to disable. `featureModels` routes side tasks (titleGeneration, compaction, tasteLearning, vision, subagents, planning/implementation lanes) to specific models; on a BYOK session model, “all the side tasks default to the session model … so nothing silently falls back to Command Code (or bills your plan) mid-session.”

## Official sources

- [Settings & Config](https://commandcode.ai/docs/settings)
- [BYOK Providers](https://commandcode.ai/docs/byok)
- [CLI Reference](https://commandcode.ai/docs/reference/cli)
- [Available Models](https://commandcode.ai/docs/reference/cli/models)
- [Permissions](https://commandcode.ai/docs/permissions)
