# Models, providers, and routing

**Verified:** 2026-11-15

## Four provider types

| Type | Access |
|---|---|
| Junie | “models accessed through a JetBrains AI subscription, by means of a Junie login or a JetBrains AI API key. No additional configuration is required.” |
| BYOK | “models accessed using your own API key from a third-party provider such as OpenAI, Anthropic, Google, xAI, OpenRouter, or GitHub Copilot” |
| Custom | “models defined in a custom profile JSON file” (or Junie Local) |
| Proxy | “models accessed through a custom proxy endpoint configured in config.json” |

“When a model is available through multiple providers, Junie prefers the Junie provider by default. Use `--provider` to override this behavior.” The `--provider`/`JUNIE_LLM_PROVIDER` values: `openai`, `anthropic`, `google`, `xai`, `openrouter`, `copilot`, `litellm`.

## Default model, aliases, and effort

“LLM model to use. If not specified, Junie uses the Default model — a dynamically set option with the best price-quality ratio.” Aliases map to “a specific model version set by the JetBrains team based on our own internal evaluations and is not necessarily the latest available version,” and “may change as new models are released.” Documented table (2026-11): `sonnet` → Claude Sonnet 5.5, `opus` → Claude Opus 5.5, `fable` → Claude Fable 5.1, `gpt`/`gpt-sol` → GPT-6.1 Sol, `gpt-codex` → GPT-5.3 codex, `gpt-astra` → GPT-6 Astra, `gpt-luna` → GPT-6 Luna, `gpt-terra` → GPT-5.6 Terra, `gemini-pro` → Gemini 3.1 Pro Preview, `gemini-flash` → Gemini 3.8 Flash, `grok` → Grok 4.7.

Effort (`--effort` / `JUNIE_EFFORT` / `/effort`) takes `minimal`, `low`, `medium`, `high`, `xhigh`, `max` per model support and “Requires JUNIE_MODEL or --model” (the CLI reference also lists `--agent`/`JUNIE_AGENT`). “We recommend keeping the default model and effort settings. Higher effort levels do not always produce noticeably better results, but they can cost significantly more.”

**Entitlement gating:** the Junie provider’s list “reflects confirmed JetBrains AI access”; with only Junie Lite confirmed, “An explicit `--model` request for a premium JetBrains model fails with an access message when only Junie Lite is available. Specify `--provider` to use the same model through another available provider.” During standalone startup, “model selection waits for the JetBrains entitlement check before automatically falling back to another provider. An explicit non-JetBrains provider selection does not wait.”

## Provider auto-detection

With `--model` but no `--provider`: the Junie provider wins when logged in; otherwise “Junie looks through the connected BYOK providers and selects the first one that offers the requested model” — e.g. `junie --model grok --grok-api-key <key>` without a JetBrains login “will automatically route to the xAI BYOK provider.”

## Primary vs faster model

The selected model is “your primary model — Junie uses it for the main reasoning and code generation work,” but internal helper tasks (summarizing context, classifying/routing, memory extraction, capability filtering) use “a separate model … an alternative from the same provider that is most suitable for these tasks.” “This applies even when using BYOK… you may see API calls to a model you didn’t explicitly select — that’s expected.”

## BYOK

Setup via `/account` (or the welcome screen). Supported: OpenAI, Anthropic, Google, xAI, OpenRouter (API keys), GitHub Copilot (OAuth token via the local GitHub Copilot CLI ≥ 1.0.65 — “Junie CLI reuses its sign-in and resolves the available models and endpoint from it”; models must be enabled in github.com/settings/copilot). Per-provider CLI flags: `--anthropic-api-key`, `--openai-api-key`, `--google-api-key`, `--grok-api-key`, `--meta-api-key` (Meta Model API / Muse Spark), `--openrouter-api-key`, `--litellm-url` + `--litellm-api-key`.

Billing: “With BYOK, Junie sends requests to LLMs using your API key directly. All usage is billed by the provider — no JetBrains AI subscription is required.” When combined with a subscription, “If a model is available through both your BYOK key and your JetBrains subscription, the BYOK key takes priority, and the requests are billed to your provider.”

Provider-specific guides: OpenRouter (`/account` → paste key), GitHub Copilot (enable in provider page; ACP can run `copilot login --web-flow` when the CLI is signed out — “Junie does not fall back to JetBrains quota”).

## Custom LLM models

JSON profiles discovered in `$JUNIE_HOME/models/*.json` and `.junie/models/*.json` (filename = profile id; select as `custom:<profile-id>`, shown in the Custom tab). Top-level fields: `id`, `baseUrl` (“the complete endpoint URL — Junie does not append a path to it”), `displayName`, `providerName`, `apiType`, `apiKey`, `extraHeaders`, `extraBody`, `temperature`, `maxContextLength`, plus `primaryModel` / `fasterModel` role objects with the same parameters (headers and extraBody merge; simple fields replace).

`apiType`: `OpenAICompletion` (Chat Completions), `OpenAIResponses`, `Google`, `Anthropic`. Secrets support `${VAR_NAME}` references “inside the `apiKey` field and inside `extraHeaders` values”; a missing variable “fails to load and Junie reports an error identifying the missing variable.” `extraBody` entries are “merged into the top level of the request JSON. If a key conflicts with a field Junie already sets (for example, `model` or `messages`), the extraBody value takes precedence, so use it with care.” Temperature: “Junie does not send a temperature value for custom models, allowing the provider to use its own default” — the docs’ recommended temperatures: DeepSeek 0, Gemini 1, Kimi 0.6–1, GLM 0.7, Qwen 0.6, MiMo 0.3 (“Setting temperature to 0 may cause looping behavior with some models”).

The custom-models page warns that agentic use needs strong models: “Smaller or heavily quantized models often cannot keep up… This is a limitation of the underlying model, not a defect in Junie,” and suggests reserving “very small or aggressively quantized variants … for the fasterModel role.”

Interactive no-profile paths exist for Ollama (`http://localhost:11434`), LiteLLM (`http://localhost:4000`), and LM Studio (`http://localhost:1234`) — Junie probes `<baseUrl>/v1/models` and sends to `<baseUrl>/v1/chat/completions` (“Do not append a path yourself”). These are CLI-only: “The Junie IDE plugin does not support Ollama or local model selection.”

## Custom proxies (EAP)

config.json `proxies` entries route LLM traffic through a self-hosted gateway; selection is `provider: "<proxy name>"` or `--provider`. Fields: `name` (required), `kind` (defaults `Ingrazzio`), `api-url` (required), `headers` (list of `Name: value`), `available-models` (filter “list is a filter, not a catalogue”), and Bedrock-only `model-overrides`. Supported kinds: **Ingrazzio** (JetBrains’ native proxy protocol; base URL serves chat, `/search`, `/extract`, `/auth/test`, `/auth/reset`), **Bedrock** (`<api-url>/model/<wire-name>/invoke`), **OpenAI**, **Google**, **Anthropic**. `JetBrainsAI` and `OpenRouter` kinds are “reserved for future use — selecting it as the active provider fails.” “When using a proxy provider, JetBrains AI authentication is bypassed entirely. You must supply any required credentials via the headers field.” Cross-file merging is by name: per-field override, headers dedup-combined, `available-models` replaced (not combined), `model-overrides` merged per id. Legacy: `INGRAZZIO_URL` env creates an implicit `ingrazzio-env` proxy that auto-becomes default “unless another provider is selected”; any config.json Ingrazzio proxy makes the env var “ignored entirely.”

## Junie Local

“A local inference engine that runs a large coding model on your own machine. It exposes an OpenAI-compatible API.” Platforms: “macOS with Apple Silicon and on Windows with an NVIDIA RTX GPU” — macOS 26+ on M4-or-newer with ≥48 GB RAM, or Windows 11 with an Ampere-or-newer RTX GPU, ≥24 GB VRAM, driver 581+; ~21 GB disk. Engines: **Splash** (MLX builds, DFlash2 draft model) on macOS; **llama.cpp with CUDA** (GGUF, MTP) on Windows. Models: Qwen3.6-27B and the JetBrains merge Qwen3.8-3.6-27B-blend, installed as 4-bit quantized builds (MLX 4-bit / Q4_K_M GGUF) — “Junie Local does not download those: it installs 4-bit quantized builds of them.” Install via `/local` or `install.sh --local-model` (~15 GB macOS / ~17 GB Windows); the installer “writes a custom model profile to `$JUNIE_HOME/models/` that points at `http://localhost:19239/v1/chat/completions`.” Junie starts/stops the engine automatically around Junie instances; `serverctl` (`~/.local/share/junie-local/current/serverctl.sh`) exists for out-of-band control (`start`, `stop`, `status`, `wait`, `health`, `uninstall <model>`, `uninstallAll`); engine updates via `/local` + Ctrl+R.

## Junie Lite

“A free baseline model tier… It is not a trial, not a limited-time sample, and it does not expire or count down.” Currently Qwen 3.6 27B — “an implementation detail, not a commitment: JetBrains may change the underlying model over time without notice… Do not build workflows around the specific model name; build around the tier.” Granted automatically on first request with any JetBrains account (also AI Pro/AI Ultimate holders); available in JetBrains Air and Junie CLI, but “Junie Lite is not available inside JetBrains AI Assistant.” “Junie Lite does not publish a quota number”; described as “virtually unlimited” for normal use; “If a model’s own capacity limit is reached, requests stop rather than falling back to another model — there is no downgrade path from Junie Lite.” Data: “Inference for Junie Lite runs exclusively on JetBrains infrastructure… your code is not used to train the underlying model.”

## Junie Eco mode

“A cost-saving run mode:” “Junie plans the task with the model you selected and implements the change with a cheaper implementation model” chosen by Junie from “the same provider route” (it “may come from a different model vendor than your planning model”). Toggle in `/model` (Space on the Eco column) or the ACP `Eco mode` session option; “For a model that supports Eco mode, Junie starts with Eco mode on until you turn it off,” persisted per model. Available only in interactive CLI and ACP — “Headless / CI runs: Eco mode is not included,” and “a prompt that continues a `/remote` bridge session does not run under Eco mode.” Intended for tasks “whose deliverable is a lasting code change in the repository”; for answers/exploration Junie may keep the selected model throughout. “Eco mode is not the same as Junie Lite.”

## Official sources

- [Model selection](https://junie.jetbrains.com/docs/junie-cli-model-selection.html) · [BYOK](https://junie.jetbrains.com/docs/byok.html) · [OpenRouter guide](https://junie.jetbrains.com/docs/byok-openrouter.html) · [GitHub Copilot guide](https://junie.jetbrains.com/docs/byok-github-copilot.html) · [Custom LLMs](https://junie.jetbrains.com/docs/custom-llm-models.html) · [Custom proxies](https://junie.jetbrains.com/docs/custom-proxies.html) · [Ollama](https://junie.jetbrains.com/docs/custom-llm-ollama.html) · [LiteLLM](https://junie.jetbrains.com/docs/custom-llm-litellm.html) · [LM Studio](https://junie.jetbrains.com/docs/custom-llm-lm-studio.html)
- [Junie Local](https://junie.jetbrains.com/docs/junie-local.html) · [Junie Lite](https://junie.jetbrains.com/docs/junie-lite.html) · [Junie Eco mode](https://junie.jetbrains.com/docs/junie-eco.html)
