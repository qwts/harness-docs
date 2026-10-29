# Configuration, precedence, and providers

**Verified:** 2026-10-29

## Shared configuration, not identical session state

The CLI and Desktop share core provider, model, extension, and general settings. Desktop exposes them through Settings and the bottom toolbar; the CLI uses `goose configure`. This does not mean an active chat immediately adopts every saved change or that every interface-specific feature is shared.

Primary paths:

| File | Purpose |
|---|---|
| `~/.config/goose/config.yaml` on macOS/Linux; `%APPDATA%\Block\goose\config\config.yaml` on Windows | Providers, model selection, extensions, and general defaults |
| `permission.yaml` | Tool permission levels configured through `goose configure` |
| `secrets.yaml` | Secrets when file-based secret storage is in use |
| `permissions/tool_permissions.json` | Automatically managed runtime permission decisions |
| `prompts/` | Custom prompt templates |

Direct configuration edits usually require a restart for existing sessions. Desktop provider saves can refresh provider inventory without restarting the app, but an already-active chat keeps the provider instance with which it started.

## Precedence is layered by feature

The general configuration order is:

1. Environment variables.
2. Config-file values.
3. Built-in defaults.

Feature-specific overrides sit around that base rather than forming one universal list:

- `GOOSE_PROVIDER` and `GOOSE_MODEL` override the provider config for that process.
- `goose run --provider ... --model ...` overrides provider/model environment settings for the run.
- Recipe `settings.goose_provider`, `settings.goose_model`, and `temperature` override ordinary defaults when that recipe executes.
- For subagents, recipe model/provider settings outrank `GOOSE_SUBAGENT_PROVIDER` and `GOOSE_SUBAGENT_MODEL`.
- Subagent turn limits resolve from the delegate call’s `max_turns`, then child-recipe `settings.max_turns`, `GOOSE_SUBAGENT_MAX_TURNS`, `GOOSE_MAX_TURNS`, and finally the default.

Do not infer that an environment variable overrides an explicit CLI flag or recipe setting merely because it wins over `config.yaml`.

## Provider selection and model capability

Current configuration uses `active_provider` plus a `providers` map. Older flat `GOOSE_PROVIDER` and `GOOSE_MODEL` config keys remain readable for compatibility and are migrated when provider settings are updated.

Goose supports hosted API providers, local OpenAI-compatible servers and local inference, ACP agents as providers, and older pass-through CLI providers. Model choice is operational, not cosmetic: the agent depends on usable tool/function calling. The provider guide explicitly excludes some models for that reason.

| Provider class | Tool and session consequences |
|---|---|
| Standard API or local model provider | Uses goose’s normal extension/tool path when the model supports it |
| ACP provider | Passes goose extensions to the external agent as MCP servers; goose session resume and fork are not currently supported |
| Deprecated CLI provider | Uses the external CLI’s own tools and filters goose extension information; does not provide goose’s MCP extension ecosystem |

The deprecated Claude Code, Codex, Gemini CLI, and Gemini OAuth providers remain for backward compatibility. Current documentation recommends ACP providers for Claude and Codex and standard Google/Vertex providers for Gemini.

### Tool shim

The experimental tool shim converts text-formatted tool calls into structured calls for models with absent or unreliable native tool calling. It uses a separate interpreter model: Ollama by default, or a configured built-in local backend. The interpreter is independent of the main conversation provider. Enabling the shim does not make every model reliable and does not change tool permissions.

## Secrets are not ordinary YAML settings

Provider API keys placed in `config.yaml` are ignored. Supported routes are:

1. The provider’s environment variable, which takes precedence over stored secrets.
2. The system keyring, configured through goose and recommended when available.
3. Plaintext `secrets.yaml` when file-based storage is selected or used as a fallback, including headless, CI, container, disabled-keyring, or inaccessible-keyring cases.

Extension `env_keys` name values to resolve when the extension starts. Resolution checks the process environment and then goose secret storage. Recipe loading does not prompt for missing values; initialization fails if a required value is unavailable. These names are not a reason to commit secret values in recipe or project configuration.

Remote extension OAuth can use dynamic registration or Client ID Metadata Documents. A pre-registered `client_id` takes priority when supplied. `client_secret_key` names a secret in extension environment/secret resolution or goose’s secret store; it is not the secret value itself.

## Extension configuration

Extensions live under the `extensions` key. Current types are:

- `builtin`
- `platform`
- `stdio`
- `streamable_http`

Deprecated SSE configuration is not supported; migrate it to Streamable HTTP. Stdio entries carry a command, arguments, environment references, and timeout. Remote entries carry a URI, optional headers/environment references, OAuth options, and timeout.

`available_tools` filters which tools goose loads from an extension. An omitted or empty list loads all tools. This reduces availability and context exposure; it is not a behavioral sandbox for the remaining tools.

For extension executables, goose constructs its command search path from `GOOSE_SEARCH_PATHS`, then built-in fallback paths, then the system `PATH`. Earlier custom directories therefore affect which executable is launched and must be trusted.

## Privacy and external processing

Anonymous usage collection is opt-in. When enabled, documented metrics include platform, goose version, provider/model, extension and tool names/counts, session metrics, and error types—not conversations, code, tool arguments, or detailed error messages. This is separate from model-provider processing: prompts, conversations, and accessed information may still go to the selected provider under that provider’s policies.

## Official sources

- [Configuration files](https://goose-docs.ai/docs/guides/config-files)
- [Environment variables](https://goose-docs.ai/docs/guides/environment-variables)
- [Provider configuration](https://goose-docs.ai/docs/getting-started/providers)
- [Installation and shared configuration](https://goose-docs.ai/docs/getting-started/installation)
- [ACP providers](https://goose-docs.ai/docs/guides/acp-providers)
- [Deprecated CLI providers](https://goose-docs.ai/docs/guides/cli-providers)
- [Tool shim](https://goose-docs.ai/docs/guides/tool-shim)
- [Using extensions](https://goose-docs.ai/docs/getting-started/using-extensions)
- [Recipe reference](https://goose-docs.ai/docs/guides/recipes/recipe-reference)
- [Anonymous usage data](https://goose-docs.ai/docs/guides/usage-data)
