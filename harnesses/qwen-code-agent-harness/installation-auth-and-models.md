# Installation, authentication, and models

**Verified:** 2026-11-16 (Qwen Code 0.27.4 npm latest; docs site main)

## Identity and install

Qwen Code is “The open-source AI coding agent for your terminal, editor, desktop, browser, and chat” (repo `QwenLM/qwen-code`, npm `@qwen-code/qwen-code`, Node.js ≥ 22). Install paths:

- **Standalone installer (recommended)** — “The recommended installer uses a standalone archive when one is available for your platform. If it falls back to npm, Node.js 22 or later with npm must be available on PATH”: `curl -fsSL https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen-standalone.sh | bash` (Linux/macOS) or the `.ps1` `irm | iex` equivalent on Windows. Offline installs use `--archive PATH` with `SHA256SUMS` alongside.
- **npm** — `npm install -g @qwen-code/qwen-code@latest`; Homebrew is also documented.

The CLI binary is `qwen`. “Qwen Code 1.0 Preview” is a separate preview line on the `v1.0.0-preview` branch (merging Qoder harness capabilities); the docs and npm latest describe the 0.27.x main line.

## First run and the five auth options

“On first launch you'll be prompted to connect a model provider.” The `/auth` menu has “five top-level options”:

1. **Qwen Account with Subscription** — “sign in with your Qwen account through **Qwen OAuth**” (browser flow); a “**Plus or Pro subscription**” from chat.qwen.ai “unlocks access to **Qwen Code and Agent**”. “For non-interactive environments where you cannot complete the browser sign-in flow, such as CI or unattended containers, use an API-key-based authentication method instead.”
2. **QwenCloud** — “official recommended setup”. Sub-menu: **Token Plan** (“subscription with a monthly Credit quota on a dedicated endpoint”; keys prefixed `sk-sp-`) or **Standard API Key** (pay-as-you-go; keys `sk-` or `sk-ws-`), then platform: **China — 千问AI平台** (`https://maas.qianwenaiapi.com/compatible-mode/v1`, Token Plan `https://token-plan.maas.qianwenaiapi.com/compatible-mode/v1`) or **International — QwenCloud** (`https://maas.qwencloudapi.com/compatible-mode/v1`, Token Plan `https://token-plan.maas.qwencloudapi.com/compatible-mode/v1`). “the UI language never decides your endpoint.” “Both access methods also enable the built-in `web_search` tool with no extra setup.”
3. **Alibaba ModelStudio** — sub-menu **Coding Plan** (individuals; weekly quota), **Token Plan** (teams; dedicated endpoint), **Standard API Key**. Coding Plan headless setup: `BAILIAN_CODING_PLAN_API_KEY` + `OPENAI_BASE_URL=https://coding.dashscope.aliyuncs.com/v1` + `OPENAI_MODEL`. Coding Plan models include “qwen3.5-plus, qwen3.6-plus, qwen3.7-plus, qwen3-coder-plus, qwen3-coder-next, qwen3-max-2026-01-23, glm-5, glm-4.7, kimi-k2.5, and MiniMax-M2.5”. Token Plan and Standard API Key also enable `web_search`.
4. **Third-party Providers** — “choose a built-in provider and connect with an API key (DeepSeek, Grok, MiniMax, Z.AI, Kimi, Idealab, ModelScope, OpenRouter, Requesty).”
5. **Custom Provider** — “manually connect a local server, proxy, or unsupported provider — supports OpenAI, Anthropic, Gemini, and other compatible endpoints.”

Environment-variable headless setup (QwenCloud example): `QWENCLOUD_API_KEY` + `OPENAI_BASE_URL` + `OPENAI_MODEL`; Token Plan uses `QWENCLOUD_TOKEN_PLAN_API_KEY`. “As with the other provider-specific keys, `QWENCLOUD_API_KEY` and `QWENCLOUD_TOKEN_PLAN_API_KEY` take effect once a `settings.json` provider entry declares them as the `envKey`; to select OpenAI-compatible auth with environment variables alone, export `OPENAI_API_KEY` instead.” `qwen auth coding-plan` has been removed.

Enterprise knobs: `security.auth.selectedType` (chosen auth), `security.auth.enforcedType` (“The required auth type (useful for enterprises)”), `security.auth.useExternal` (external auth flow). “QwenCloud and Alibaba ModelStudio are separate entries with separate env keys and endpoints. Choosing QwenCloud does not rewrite or delete saved ModelStudio credentials.” Startup checks the endpoint's model list with the stored key (“Built-in provider updates”).

## `modelProviders` configuration

`modelProviders` maps each provider id to “**an array of model definitions** (`ModelConfig[]`). For built-in providers the key must be a valid auth type (`openai`, `anthropic`, `gemini`, `vertex-ai`); a custom provider id (e.g. `idealab`) is allowed as long as you map it to a protocol via the top-level `providerProtocol` setting.” Each entry requires `id`; “`envKey` is **optional and recommended** (when omitted, it falls back to the auth type's default env key, e.g. `OPENAI_API_KEY` for `openai`)”, with optional `name`, `description`, `baseUrl`, `generationConfig`, and `wireApi` (`chat-completions` vs `responses`). “Credentials are never persisted in settings.”

```json
{
  "modelProviders": {
    "openai": [
      {
        "id": "qwen3.8-max",
        "name": "qwen3.8-max (QwenCloud)",
        "baseUrl": "https://maas.qwencloudapi.com/compatible-mode/v1",
        "envKey": "QWENCLOUD_API_KEY"
      }
    ]
  },
  "security": { "auth": { "selectedType": "openai" } },
  "model": { "name": "qwen3.8-max" }
}
```

Behaviors worth knowing:

- “Models are identified by their effective API protocol, `id`, and configured `baseUrl` … If entries share all three values, the first occurrence wins and subsequent duplicates are skipped with a warning.”
- “`modelProviders` edits in `settings.json` are picked up by a running interactive session without a restart (the file watcher debounces ~300ms; reopen `/model` to see new entries …). Changing the active model's `wireApi` creates a different route … `providerProtocol` is read once at startup and **requires a restart**.”
- “`capabilities` is the one provider stamp that is never inferred from the model name” — a hand-written entry “resolves with no reasoning declaration and `/effort` offers the generic tier list rather than QwenCloud's curated `low` / `medium` / `xhigh`.”
- Image generation routes: `supportsImageGeneration: true` enables the built-in `image_gen` tool; `imageOnly: true` hides the route from ordinary selectors.
- The earlier wrapped shape `{ "protocol": ..., "models": [...] }` “has been reverted — the current value is the bare `ModelConfig[]` array … A wrapped entry in an already-migrated (`$version: 4`) settings file is silently skipped.”

Protocol-specific pages document OpenAI-compatible providers, the OpenAI Responses API, Anthropic, Google Gemini, OCI Generative AI, Wally/RunAnywhere, Wiro AI, and “Local Self-Hosted Models (via OpenAI-compatible API)” (Ollama/vLLM examples with empty API keys).

## Model slots, catalog, and selection

- Multiple named model slots in settings: `model` (main), `fastModel` — “Model used for generating prompt suggestions and speculative execution. Leave empty to use the main model.” The Auto-mode classifier, session recap (“as a side-query against the fast model”), and tool-use summaries (“Requires a fast model to be configured (`fastModel`)”) also use it, `advisorModel` (`/advisor` second opinion), `visionModel`, `compactionModel`, `imageModel`, `voiceModel`, plus `modelFallbacks` and `modelPricing`.
- **models.dev catalog**: `QWEN_CODE_MODELS_DEV=off` tells Qwen Code to “ignore the bundled models.dev model catalog and resolve context windows, output limits, and input modalities from the built-in model-name tables only.” A daily refresh downloads into `~/.qwen/model-registry.json` (`QWEN_CODE_MODELS_DEV_REFRESH=off` stops it; `QWEN_CODE_MODELS_DEV_URL` mirrors it). “Project-level `.env` files and `settings.json` `env` cannot set this key.”
- `/model` switches models (including provider routes); `--model`/`-m` sets one for a session; `/effort` manages reasoning effort (QwenCloud “curated `low` / `medium` / `xhigh`”).
- “Legacy free Qwen OAuth models remain hard-coded and cannot be overridden.”
- Output token limits: `QWEN_CODE_MAX_OUTPUT_TOKENS` “Overrides the default maximum output tokens per response … Set this to a specific value (e.g., `16000`) to use a fixed limit”; otherwise the CLI “defaults to the model's declared output limit and, if a response is truncated, automatically escalates (64K floor) and recovers across turns,” with `samplingParams.max_tokens` taking precedence.

## Official sources

- [Overview](https://qwenlm.github.io/qwen-code-docs/en/users/overview/) · [Quickstart](https://qwenlm.github.io/qwen-code-docs/en/users/quickstart/) · [Authentication](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/auth/) · [Model Providers](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/model-providers/) · [Settings](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/) · [GitHub README](https://github.com/QwenLM/qwen-code)
