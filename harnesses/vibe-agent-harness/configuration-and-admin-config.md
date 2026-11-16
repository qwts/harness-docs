# Configuration and admin config

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site main)

## config.toml

“The Vibe Code CLI is configured through a **`config.toml` file**.” Merge model: “The CLI merges two `config.toml` files: 1. **Project-level**: the closest `.vibe/config.toml`, found by walking up from the working directory, without going past your home directory. 2. **User-level**: `~/.vibe/config.toml`. Project settings override the same user settings … Project configuration is loaded only when its folder is [trusted].”

`/config` “saves changes to the user file by default. Use **Save to** in the settings editor to save to the project file or to the current session only.” Validation: “At startup, Vibe warns about settings it doesn't use, such as a misspelled key … To refuse to start instead, for example in CI, set `strict_config = true` or `VIBE_STRICT_CONFIG=1`.” “`vibe config schema` prints a JSON schema of every setting.”

**Vibe home**: “the CLI stores its configuration and state in `~/.vibe/`. Override the location with the `VIBE_HOME` environment variable,” which moves `config.toml`, `.env` (credentials), `agents/`, `prompts/`, `skills/`, `plugins/`, and `logs/`. “Vibe no longer loads custom Python tools from `~/.vibe/tools/` or `./.vibe/tools/`, and shows a warning at startup for each file there.”

## Selected top-level keys

`default_agent` (`"accept-edits"`), `active_model`, `allowed_models`, compaction keys (`compaction_model`, `auto_compact_threshold`, `compaction_max_output_tokens`, `compaction_reasoning_limit_tokens` 32000), `agent_max_output_tokens` (0), `output_limit_retries` (6), `vision_model`, agent/skill/tool filters (`enabled_agents`/`disabled_agents`, `agent_paths`, `system_prompt_id` `"cli"`, `compaction_prompt_id` `"compact"`, `title_prompt_id`, `skill_paths`, `enabled_skills`/`disabled_skills`, `enabled_tools`/`disabled_tools`), `outside_workdir` (`"ask"`), capability switches (`enable_mcp` true, `enable_connectors` true, `enable_auto_approve` true, `enable_subagents` true, `max_subagents` 32, `max_concurrent_subagents` 32, `enable_background_processes` true, `enable_update_checks` true, `enable_notifications` true, `enable_telemetry` true, `enable_system_trust_store` false), UX keys (`ask_confirmation_on_exit` true, `enable_mouse_capture` true, `show_thinking_nodes` true, `enable_file_autocomplete` true, `file_watcher_for_autocomplete` true, `theme` `"auto"`), worktree keys (`worktree_limit` 15, `worktree_copy_on_write` true — “macOS only … Sets `extensions.worktreeConfig`”), `log_level` `"WARNING"` (`~/.vibe/logs/vibe.log`), `strict_config` false, experiments (`experimental_enable_mistral_mcp_connector` false, `experimental_enable_model_catalog` false), plus `[[providers]]`, `[[models]]`, `[[mcp_servers]]`, `[utility_models]` (“the smart approve classifier and session titles”), and `[session_logging]`.

“An exhaustive index of every key accepted by `config.toml` and the agent-definition files” lives in the configuration reference page.

## Admin config (managed config)

“Admin config, also called **managed config**, lets Organization and Workspace administrators distribute settings for the Vibe Code CLI and the VS Code extension to all users … Use it to distribute models, providers, MCP servers, and tool policies without requiring users to edit `config.toml`.”

- **Not a security control**: “Admin config is a **distribution mechanism, not a security control** … users who control their local CLI environment can bypass them. Use it to set safe defaults, not to guarantee access rules: anything users must never reach also needs a server-side control.”
- **Delivery**: “The CLI fetches it over HTTP and keeps a copy in `~/.vibe/managed_config_cache.json`. It never writes admin config to `config.toml`.” “The CLI checks for changes every 10 minutes in the background. `/reload` fetches it right away.” Sessions start on the cached copy and “waits for the endpoint … only when there's no copy yet.”
- **Scopes, no merging**: “The server doesn't merge the two configurations.” Workspace choices: inherit Organization, “Enabled” (Workspace-only — “Settings defined only at the Organization level don't apply”), or “Disabled” (“No admin config, even when the Organization has one”).
- **Precedence**: “If admin config is enabled, its values take precedence over every other configuration source.”
- **Fallback**: “When admin config can't be applied, for example because it's invalid, sessions run on the user's local configuration instead.”
- **Gating**: requires the `manage_code_config` RBAC permission at Organization scope; `enable_mcp = false`, `enable_connectors = false`, and `enable_auto_approve = false` block MCP, connectors (including built-in web search), and auto-approve respectively.
- **Minimum versions**: CLI `2.24.2`, VS Code extension `1.16` — “Earlier clients don't request admin config or apply its settings.”

## Official sources

- [Configuration](https://docs.mistral.ai/vibe/code/cli/configuration) · [Configuration reference](https://docs.mistral.ai/vibe/code/cli/configuration-reference) · [Admin config](https://docs.mistral.ai/vibe/code/cli/admin-config) · [API keys and profiles](https://docs.mistral.ai/vibe/code/cli/api-keys-profiles)
