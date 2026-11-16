# Installation, auth, and models

**Verified:** 2026-11-16 (Pi v1.5.2)

## Install and update

Three first-party install paths:

- **Installer** (macOS/Linux/Windows): `curl -fsSL https://pi.dev/install.sh | sh`, or on Windows `powershell -c "irm https://pi.dev/install.ps1 | iex"`. “Both installers pin all dependencies and update Pi with `pi update`.”
- **npm**: `npm install -g --ignore-scripts @earendil-works/pi-coding-agent`, “This requires Node.js 22.19 or newer,” and “Pi does not require dependency lifecycle scripts for a normal npm installation.”
- **Nix** (macOS/Linux): `nix profile add github:earendil-works/pi/stable` (older Nix: `nix profile install`); “Update with `nix profile upgrade pi`; `pi update` cannot update a Nix installation. To pin a release, use a tag such as `github:earendil-works/pi/v1.0.0`.” `pi update` generally “cannot update Pi when another package manager provides it”; wrappers can set `PI_SELF_UPDATE_COMMAND` or the `selfUpdateCommand` setting (an argv array), whose `{packageName}`, `{version}`, and `{installSpec}` placeholders are filled from the newer pi.dev release.

Verify with `pi --version`. Uninstall paths (npm uninstall, installer **Uninstall Pi**, `nix profile remove pi`) share one limit: “None of these methods removes configuration, credentials, sessions, or installed Pi packages from `~/.pi/agent/`.”

## Credential resolution

“Run `/login` and select a provider. Pi stores credentials in `auth.json`. Run `/logout` to remove stored credentials for a provider.” Precedence when several sources exist: “Pi uses a runtime `--api-key` first, then a stored `auth.json` credential, an `apiKey` from `models.json`, and finally the provider's environment variables or ambient cloud credentials. Provider extensions can define their own authentication behavior.”

Secret managers stay out of disk via command indirection: “set a provider's `key` in `auth.json` to a command prefixed with `!`” — “Pi runs the command when the key is first needed and caches its standard output for the process lifetime. Empty output, a timeout, or a nonzero exit leaves the key unresolved until Pi restarts.” A stored credential may also carry an `env` object — “Its values take priority over the process environment for that provider” (used for Cloudflare account IDs and similar). Keep `auth.json` private: “`auth.json` can contain API keys and OAuth tokens. Keep it private and do not commit it.”

The `pi auth` subcommands operate without a session: `pi auth login <provider>` (`--api-key` saves a key read without echoing; `--no-browser` uses device codes over SSH and is “Not allowed with `--api-key`”), `pi auth logout` (“exits with status `0` when nothing is stored”), and `pi auth check` / `print-api-key` / `print-bearer-token` (which “write secrets to stdout”; `check` exits `0`/`1`/`2` for `ready`/`not_ready`/`invalid` and supports `--json`, `--credentials`, `--no-refresh`, `--min-expiry`).

## Providers

Most providers support OAuth sign-in or an API key; “Amazon Bedrock and Google Vertex AI can also use ambient cloud credentials.” Single-variable providers:

| Provider | Env var | Provider | Env var |
|---|---|---|---|
| Anthropic | `ANTHROPIC_API_KEY` | Radius | `RADIUS_API_KEY` |
| Ant Ling | `ANT_LING_API_KEY` | TypeSafe (classifiers) | `TYPESAFE_API_KEY` |
| OpenAI | `OPENAI_API_KEY` | Hugging Face | `HF_TOKEN` |
| DeepSeek | `DEEPSEEK_API_KEY` | Fireworks | `FIREWORKS_API_KEY` |
| NVIDIA NIM | `NVIDIA_API_KEY` | Together AI | `TOGETHER_API_KEY` |
| Google Gemini | `GEMINI_API_KEY` | Baseten | `BASETEN_API_KEY` |
| GitHub Copilot | `COPILOT_GITHUB_TOKEN` | Kimi For Coding | `KIMI_API_KEY` |
| Mistral | `MISTRAL_API_KEY` | Meta | `META_API_KEY` |
| Groq | `GROQ_API_KEY` | MiniMax / MiniMax (China) | `MINIMAX_API_KEY` / `MINIMAX_CN_API_KEY` |
| Cerebras | `CEREBRAS_API_KEY` | Moonshot AI (Global and China) | `MOONSHOT_API_KEY` |
| xAI | `XAI_API_KEY` | Qwen Token Plan Team / (China) | `QWEN_TOKEN_PLAN_TEAM_API_KEY` / `…_CN_API_KEY` |
| OpenRouter | `OPENROUTER_API_KEY` | Qwen Token Plan Individual / (China) | `QWEN_TOKEN_PLAN_INDIVIDUAL_API_KEY` / `…_CN_API_KEY` |
| Vercel AI Gateway | `AI_GATEWAY_API_KEY` | Xiaomi MiMo | `XIAOMI_API_KEY` |
| ZAI Coding Plan (Global / China) | `ZAI_API_KEY` / `ZAI_CODING_CN_API_KEY` | Xiaomi MiMo Token Plan (China/Amsterdam/Singapore) | `XIAOMI_TOKEN_PLAN_CN_API_KEY` / `…_AMS_API_KEY` / `…_SGP_API_KEY` |
| OpenCode Zen and Go | `OPENCODE_API_KEY` | | |

Provider-specific notes: Anthropic also recognizes `ANTHROPIC_OAUTH_TOKEN` (API credential) and `ANTHROPIC_AUTH_TOKEN` (bearer), plus workload identity federation via `ANTHROPIC_FEDERATION_RULE_ID`/`ANTHROPIC_ORGANIZATION_ID`/`ANTHROPIC_IDENTITY_TOKEN_FILE`. **Azure**'s provider ID is `azure` (formerly `azure-openai-responses`) with `AZURE_OPENAI_API_KEY` + `AZURE_OPENAI_BASE_URL` or `AZURE_OPENAI_RESOURCE_NAME`, `AZURE_OPENAI_DEPLOYMENT_NAME_MAP`, and `AZURE_OPENAI_API_VERSION` (default `v1`); Foundry models route through Chat Completions. **Bedrock** uses `AWS_BEARER_TOKEN_BEDROCK` or standard AWS credential chains (profiles, IAM keys, ECS, IRSA). **Cloudflare** needs `CLOUDFLARE_API_KEY` + account ID (+ gateway ID for AI Gateway). **Vertex AI** takes `GOOGLE_CLOUD_API_KEY` or Application Default Credentials with `GOOGLE_CLOUD_PROJECT`/`GOOGLE_CLOUD_LOCATION`. **Radius** (“a service crafted for Pi by the builders of Pi, Earendil Works”) is “currently in early alpha”; logging in to Radius also adds the Radius MCP server to `~/.pi/agent/mcp.json` when no global server uses that login.

## Model selection

“Run `/model` to search available models. The picker shows models whose providers have usable authentication. Press `Ctrl+S` on a model to save it as the default for new sessions.” `/thinking` sets the level (“Pi limits the choices to levels supported by the selected model”); `Ctrl+P` cycles through models, with the cycle configured by `/scoped-models`. “A session records model and thinking-level changes. Resuming the session restores them without changing defaults for new sessions.”

Launch flags: `--model <pattern>` (“exact ID or fuzzy ID/name match … accepts `provider/id` and an optional `:<thinking>` suffix. If several providers match, Pi prefers the one with credentials”), `--provider <name>` (“Restricts `--model` lookup to one provider. It requires `--model`”), `--api-key <key>` (non-persistent override requiring a model selection), `--thinking <level>` (“Sets `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, or `max`. It overrides a `--model` suffix and is clamped to the model's capabilities”), `--models <patterns>` (comma-separated scope for startup and cycling, with case-insensitive globs), and `--list-models [search]`. Settings defaults: `defaultProvider`, `defaultModel`, `defaultThinkingLevel` (default `"medium"`), `modelThinkingLevels` per exact `provider/modelId`, `enabledModels`.

The catalog: “Pi starts with its bundled catalog and can overlay newer catalog data from pi.dev. Cached catalog data remains available offline; run `pi update --models` to force a refresh.”

## Compatible endpoints (models.json)

`~/.pi/agent/models.json` adds endpoints that speak an API Pi supports — “most Ollama, LM Studio, vLLM, SGLang, and proxy deployments”:

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "models": [{ "id": "qwen2.5-coder:7b" }]
    }
  }
}
```

Key semantics: “`apiKey` and header values can use `$NAME` or `${NAME}` environment interpolation, a literal value, or a leading `!command`. `$$` and `$!` produce a literal `$` and `!`. Commands in `models.json` run at request time and are not cached by Pi.” “Opening `/model` reloads the file. A `models` entry adds or replaces a model with the same ID on that provider. Use `modelOverrides` to change metadata for an existing built-in or extension-provided model without replacing the provider's model list. Unknown override IDs are ignored.” Proxying a built-in provider needs only `baseUrl` — “Its models and authentication stay unchanged.”

Model entries can describe input and caching: `inputLimits.images.resize` (defaults “2000 by 2000 pixels, 4.5 MiB encoded, and JPEG quality 80”; “Images are encoded once; changing models does not rewrite historical images”), `promptCache` short/long lifetimes for cache warming, and `samplingParams`/`samplingParamsByThinkingLevel` keyed by Pi thinking levels — “These fields apply only to `openai-completions`, `openai-responses`, and `azure-openai-responses`.” The docs caution: “Compatibility settings should describe verified differences in the endpoint's request or response behavior. Do not enable them based only on an endpoint advertising OpenAI or Anthropic compatibility.”

## Local models, classifiers, and image models

“Pi integrates directly with the llama.cpp router. The router discovers GGUF files and loads models on demand. Pi's `/llama` command manages the router, while `/model` selects one of its loaded models.” Other local servers (Ollama, LM Studio, vLLM, SGLang) go through `models.json`.

Classifier models (“do not chat … answer typed questions about JSON state … with probabilities”) include TypeSafe's Jev, Cloudflare's Clef family, OpenAI's `gpt-6-luna` (Decisions API), and `microsoft-decision-1` via GitHub Copilot. Image models (OpenRouter's, such as `google/gemini-2.5-flash-image`) generate images. “Classifier models do not appear in `/model`” — nor do image models; the model reaches them through the `codemode` tool (`models.classify(...)`, `models.generateImages(...)`), and extensions call them through `ctx.modelRegistry.classify()` / `.generateImages()`.

## Provider extensions and virtual models

When a service needs custom streaming, discovery, or auth, a provider extension registers through `pi.registerProvider()` — redirect an existing provider with only `baseUrl`/`headers` (“Do not copy the built-in model list into the extension”), or register a full compatible endpoint. A **virtual model** is “a selectable model that picks a physical model for each request” via `pi.registerVirtualModel({ provider, id, thinkingLevels, route(request, ctx) })`; “Providers only receive physical models. Assistant messages name the physical model,” and the footer shows “the routed model next to the selection, for example `auto • high → gpt-5.6-luna • medium`.”

## Official sources

- [Quickstart](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/quickstart.md) · [Choose a Model](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/models.md) · [Providers](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/providers.md) · [Command Line](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md) · [Custom Providers](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/custom-provider.md) · [Virtual Models](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/virtual-models.md) · [llama.cpp](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/llama-cpp.md)
