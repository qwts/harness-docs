# Models, BYOK, and custom routing

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main)

## Choosing a model

“Run `/model` to open the model picker. You can also click the model name in the statusline to open the same picker.” “Models that your plan or your team's settings don't allow appear disabled in the picker.” “Your choice is saved as the base model of your active profile, so it persists across sessions.”

Warp curates models from OpenAI, Anthropic, Google, and open-weight providers, addressed by `model_id`:

- **Auto models** — `auto` (“Selects the highest-quality, fastest available model”), `auto-efficient` (“Optimizes for lower inference cost while maintaining output quality”), `auto-genius` (“Adapts to task complexity and selects Warp's most capable model when it's worth it. Best for deep debugging, architecture decisions, and `/plan` sessions”), and `auto-open` (“Routes between the best open-source models available in Warp”).
- **OpenAI** — reasoning-level suffixes (`-low` … `-max`) on GPT-6/6.1/5.x families (e.g. `gpt-6-astra-max`, `gpt-6-1-sol-high`).
- **Anthropic** — effort suffixes on Claude Opus/Sonnet/Fable/Haiku families (e.g. `claude-5-5-opus-xhigh-fast`, `claude-4-7-opus-max`); `thinking` suffixes on 4.5-era IDs. ZDR caveat: “Anthropic requires data retention for Claude Fable 5, Claude Fable 5.1, and Claude Fable 5.2 for safety, abuse monitoring, and compliance reasons, so these models are **not available under Zero Data Retention (ZDR)**.”
- **Google** — `gemini-4-argon`, `gemini-3.1-pro`, `gemini-3.x-flash`.
- **xAI** — Grok models, usable via a connected X Premium/SuperGrok subscription.

The same catalog powers cloud runs: “Choose from the models and routers available in the model picker” — the Warp Agent is Warp's “built-in harness for factory agents and standalone cloud runs.”

## Bring Your Own API Key (BYOK)

“Warp supports **Bring Your Own API Key (BYOK)** for users who want to connect agents to their own Anthropic, OpenAI, or Google API accounts. … Your provider bills inference routed through your keys. Applicable platform charges remain.”

- **Storage** — “Personal API keys are stored in your device's secure storage.” “Warp never stores provider keys on its servers,” and “Warp does not retain these personal provider keys on its servers” is repeated for the CLI menu.
- **CLI management** — `/api-keys` lists providers with connection state plus a Warp-credit fallback toggle; “Input is masked, so the key never appears in your transcript or shell history.” Outside a session: `warp --set-provider-api-key <provider>` / `warp --clear-provider-api-key <provider>` (`openai`, `anthropic`, or `google`); piped stdin is supported (“so you can pass it straight from a secret manager without writing it to disk”).
- **Separation** — “The CLI and Warp app store provider keys separately. Configure a key in each surface you use.”
- **Subscriptions** — ChatGPT subscriptions cover eligible OpenAI models; “Connect X Premium or SuperGrok” via `/api-keys` or `/connect-grok`. “You can't connect a Claude consumer subscription to the Warp Agent. To use your own Anthropic account, configure BYOK.”
- **Plan limits** — “BYOK is available on Free and all eligible paid plans for individual users and organizations with 10 or fewer employees … Larger organizations need a Business or Enterprise plan.”

Billing behavior: “When you select a model with a key icon, Warp routes inference through your own API key instead of consuming Warp-provided inference usage” and “Inference is billed through your provider account rather than your Warp usage balance.” Failover: “**Warp credit fallback** - Press `Enter` to toggle whether a request that fails through your key is retried with Warp-provided inference, consuming your available usage.”

## Custom routers

“Custom routers appear in the `/model` picker alongside individual models. Select one and each prompt resolves to a concrete model using the routing logic you defined.” “The CLI loads the same router definitions as the Warp app. These are YAML files in `~/.warp/custom_model_routers/`, plus any team-synced routers your admin shares.” Routers can also be authored against your own keys: a router “whose targets your keys cover” bills inference through your provider accounts.

Distinct enterprise paths: **Personal custom inference endpoint** (“an OpenAI-compatible endpoint such as OpenRouter, LiteLLM, z.ai, or an internal gateway”), **Team-managed API keys and endpoints** (Business/Enterprise — “which Warp stores securely for local and cloud inference”), and **BYOLLM** (Enterprise — “Route inference through your organization's AWS Bedrock or Gemini Enterprise (Vertex AI) account”).

## Usage and cost

“The CLI shows usage at three levels”: `/usage` for account-level “plan, credit balances and consumption, and links to manage billing or upgrade”; the statusline “Conversation usage” chip (“Accounts billed in credits always see credits”); and `/cost` for “the latest completed response's elapsed time and usage.” `/copy-debugging-id` copies conversation debugging information for bug reports.

## Official sources

- [Model choice](https://docs.warp.dev/agents/inference/model-choice) · [Bring Your Own API Key](https://docs.warp.dev/agents/inference/bring-your-own-api-key) · [Custom routers](https://docs.warp.dev/agents/inference/custom-routers) · [ChatGPT subscription](https://docs.warp.dev/agents/inference/chatgpt-subscription) · [Grok subscription](https://docs.warp.dev/agents/inference/grok-subscription) · [Models and usage in the CLI](https://docs.warp.dev/agents/cli/models-and-usage)
