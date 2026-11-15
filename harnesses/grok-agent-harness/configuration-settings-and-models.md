# Configuration, settings, and models

**Verified:** 2026-11-15

## The `~/.grok/` layout

Configuration is TOML under `~/.grok/` (or `$GROK_HOME`), not JSON: “Settings are persisted under `~/.grok/config.toml` (on Windows, `%USERPROFILE%\.grok\config.toml`). To configure the default home directory, you can set `$GROK_HOME`.” The reference lists `GROK_HOME` as “Home for config, auth, sessions, skills, plugins, and logs.”

## Five scopes — and what project scope may contain

| Scope | Path | Use for |
|---|---|---|
| Environment | `GROK_*` (and related) variables | Session / CI overrides |
| User | `~/.grok/config.toml` (or `$GROK_HOME/config.toml`) | Personal defaults |
| Project | `.grok/config.toml` in the repo | Repo-shared MCP, plugins, and permission rules |
| Managed | `~/.grok/managed_config.toml`, `/etc/grok/managed_config.toml` | Enterprise-served defaults |
| Requirements | `~/.grok/requirements.toml`, `/etc/grok/requirements.toml` | Policy pins |

The reference is explicit about project limits: “Project `.grok/config.toml` only contributes **`[mcp_servers]`**, **`[plugins]`**, and **`[permission]`**. Other sections belong in user config.” The settings page adds: “Project configs are limited to MCP servers, plugins, and permission rules, not full user configs.”

Enterprise layering (lowest → highest): `/etc/grok/managed_config.toml` → `~/.grok/managed_config.toml` → `~/.grok/config.toml` → `~/.grok/requirements.toml` → `/etc/grok/requirements.toml`. “Settings in `requirements.toml` cannot be overridden by lower layers, remote settings, or user config — use it for compliance-critical policies,” and all layers support `[[version_overrides]]` for version-conditional patches plus `$VAR` expansion. `grok inspect [--json]` shows “the configuration Grok discovers for this directory: rules, skills, plugins, hooks, and MCP servers”; `grok setup` fetches managed configuration.

## Models and BYOK

`[models]` holds routing keys: `default` (the docs recommend `grok-build` for coding sessions), `web_search` (the client-side `web_search` tool’s model), `default_reasoning_effort`, `session_summary`, `image_description`, global `temperature`/`top_p`/`max_completion_tokens`, `max_retries`, `stream_tool_calls` (“some BYOK endpoints need `false`”), `extra_headers` (per-model keys win), and picker controls: `allowed_models` (glob list restricting picker/default/`-m`), `hidden_models` (“still usable via `-m`”), and `disabled_models` (“wins over hidden”).

Custom models are `[model.<id>]` tables — “Custom / BYOK models (OpenAI-compatible or Anthropic Messages). Prefer `env_key` over hardcoding `api_key`” — with `model`, `base_url`, `name`, `description`, `api_key`/`env_key`, `api_backend` (`chat_completions` | `responses` | `messages`), sampling, `context_window` (“drives auto-compact timing”), `extra_headers`, `supports_backend_search`, reasoning controls, and `max_retries`/`inference_idle_timeout_secs`. Credential resolution is per model: “`model.api_key` > `model.env_key` > active session token > `XAI_API_KEY`.” Env vars reroute inference wholesale: `GROK_MODELS_BASE_URL` (“model list from `{base}/models`”), `GROK_MODELS_LIST_URL`, `GROK_XAI_API_BASE_URL` (default `https://api.x.ai/v1`), `GROK_DEFAULT_MODEL`, and `GROK_WEB_SEARCH_MODEL`.

## Key environment variables

- **Paths/auth:** `GROK_HOME`, `XAI_API_KEY` (“API key when not using browser/session login (CI and headless)”).
- **Sandbox:** `GROK_SANDBOX` (`off` | `workspace` | `read-only` | `strict` | custom; same as `--sandbox`), `GROK_SANDBOX_AUTO_ALLOW_BASH`.
- **Feature toggles:** `GROK_WEB_FETCH` (`0` default — “Off by default for security”), `GROK_MEMORY` (`0`), `GROK_SUBAGENTS` (`0`), `GROK_GOAL` (`1`; “Set `0` to remove `/goal`”), `GROK_AGENT` (default `grok-build`), `GROK_WRITE_FILE` (`1`; `0` = read-only sessions), `GROK_TOOL_SEARCH` (`1` — on-demand MCP tool discovery), `GROK_LSP_TOOLS` (`0`), `GROK_RESPECT_GITIGNORE`.
- **MCP/ops:** `GROK_MCP_STARTUP_TIMEOUT_SECS` (`30`; per-server `startup_timeout_sec` wins), `MCP_TIMEOUT` (Claude-compatible, milliseconds, “checked before” the Grok one), `GROK_LOG_FILE`, `RUST_LOG`, `GROK_CRASH_HANDLER` (reports under `$GROK_HOME/crash/`), standard `HTTPS_PROXY`/`HTTP_PROXY`/`NO_PROXY`.
- **Update control:** `GROK_DISABLE_AUTOUPDATER`, plus `[cli] auto_update` and `--no-auto-update` for headless/ACP.

## Compatibility scanners

Grok scans Claude Code and Cursor harness directories by default, each with an on/off env var (`GROK_CLAUDE_SKILLS_ENABLED`, `GROK_CLAUDE_RULES_ENABLED`, `GROK_CLAUDE_AGENTS_ENABLED`, `GROK_CLAUDE_MCPS_ENABLED`, `GROK_CLAUDE_HOOKS_ENABLED`; the five `GROK_CURSOR_*` equivalents). TOML mirrors them under `[compat.cursor]` / `[compat.claude]` with `skills` / `rules` / `agents` / `mcps` / `hooks` keys, “`true` / `false` (default `true`)”. The skills page states the outcome: “Grok is fully compatible with Claude Code with zero configuration needed,” reading “marketplaces, plugins, skills, MCPs, agents, hooks, and instruction files (`CLAUDE.md`, `Claude.md`, `CLAUDE.local.md`, and `.claude/rules/`) alongside `.grok/`.”

## Notable TOML sections

- **`[permission]`** — compact `allow`/`deny`/`ask` rule arrays (“Evaluation order: **deny > ask > allow**”; examples `Bash(git *)`, `Read(src/**)`, `MCPTool(server__*)`) or verbose `rules = [{ action, tool, pattern? }]` with `tool`: `any` | `bash` | `edit` | `read` | `grep` | `mcp` | `webfetch`.
- **`[sandbox]`** — `profile` (`off` default) and `auto_allow_bash`; custom profiles live in `~/.grok/sandbox.toml` or `.grok/sandbox.toml` (built-in names cannot be redefined).
- **`[tools]` / `[toolset.*]`** — `respect_gitignore`, `disable_zdr_incompatible_tools`, `[tools.zdr_video_output_s3]`, `file_toolset` (`standard` | `hashline`), `[toolset.bash]` `timeout_secs` (120) / `output_byte_limit` (20000) / `max_timeout_secs` (36000), `[toolset.web_fetch]` `proxy_endpoint` / `allowed_domains`.
- **`[session]`** — `auto_compact_threshold_percent` (85), `load_envrc` (true — injects `.envrc` into bash).
- **`[ui]`** — `permission_mode` (`default`/`ask`/`auto`/`always-approve`; “Enterprise locks use `requirements.toml`”), `default_selected_permission` (`always_allow_all_sessions` default), `theme` (default Grok Night; `auto` follows OS via `auto_dark_theme`/`auto_light_theme`), `screen_mode` (`fullscreen` default | `minimal`), `simple_mode`/`vim_mode`, voice keys, `status_line` (default `disabled`; `builtin`/`command`/`disabled`), `disable_bypass_permissions_mode` (enterprise lock).
- **`[hints]`** — `new_session_worktree_mode` (`never` default) and `fork_worktree_mode` (`ask` default) control whether `/new` and `/fork` offer a worktree.
- **`[features]` / `[subagents]` / `[memory]` / `[goal]`** — `web_fetch`, `lsp_tools`, `write_file`, `tool_search`; `[subagents] enabled` plus `[subagents.toggle]` and `[subagents.models]` maps; `[memory] enabled` (default off); `[goal] enabled` (default on).

## Official sources

- [Settings](https://docs.x.ai/build/settings) · [Settings Reference](https://docs.x.ai/build/settings/reference)
- [Overview](https://docs.x.ai/build/overview) · [CLI Reference](https://docs.x.ai/build/cli/reference)
- [Enterprise Deployments](https://docs.x.ai/build/enterprise) · [Permissions](https://docs.x.ai/build/features/permissions) · [Sandbox](https://docs.x.ai/build/features/sandbox)
