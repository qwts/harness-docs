# Configuration, providers, and models

**Verified:** 2026-11-15

## Config file: one `kilo.jsonc` system, many locations

Kilo Code is configured through a single JSON(C) config key-space read from several locations. The CLI docs state the recommended file is `kilo.json` and also support `kilo.jsonc` and `config.json`:

| Scope | Recommended path | Also supported |
|---|---|---|
| Global | `~/.config/kilo/kilo.json` | `kilo.jsonc`, `config.json` |
| Project | `./kilo.json` or `./.kilo/kilo.json` | `kilo.jsonc` |

“Project-level configuration takes precedence over global settings.” Config files support a `$schema` pointing at `https://app.kilo.ai/config.json`, comments (`jsonc`), and `{env:VAR}` substitutions that expand environment variables at load time. The `KILO_CONFIG_CONTENT` environment variable supplies config content directly, overriding file-based configuration.

Agent-level configuration precedence (lowest to highest): built-in defaults → global `kilo.jsonc` → project `kilo.jsonc` → `.kilo/` directory configs plus agent `.md` files → `KILO_CONFIG_CONTENT`.

Reload behavior differs by scope: project config is cached when the workspace loads (a window reload is needed to pick up edits), while global config changes auto-reload.

## Authentication and accounts

- **Kilo account** — sign in via the extension/CLI to use the Kilo Gateway with Kilo credits; account management lives at app.kilo.ai.
- **Gateway API keys** — “API keys are JWT tokens tied to your Kilo account,” sent as `Authorization: Bearer <your_api_key>`; organization-scoped requests add `X-KiloCode-OrganizationId: your_org_id`. Keys stored by the gateway “are encrypted at rest using AES-256 encryption.”
- **BYOK** — provider keys can be attached instead of (or alongside) Kilo billing. Routing rule: “If a matching BYOK key exists and the request is not served by a connected subscription like ChatGPT, the request is routed using your key.” Failure behavior is explicit: “If the key is invalid, the request fails. It does not fall back to using Kilo's keys.” Subscription-based providers — “such as the Z.ai Coding Plan or Kimi Code” — “only expose the models included in that plan.”
- **Anonymous** — the gateway’s free models work without an account, rate-limited to 200 requests per hour per IP address.

## The Kilo Gateway

The gateway is Kilo’s OpenAI-compatible proxy at `https://ai-gateway.kilo.ai/api/v1`. Models are addressed as `provider/model-name` (e.g. `anthropic/claude-sonnet-4.6`); `GET /api/v1/models` lists the catalog including pricing and context windows — “No authentication is required.” Model IDs can be suffixed `:free` for zero-cost variants.

Billing tracks “microdollar precision (1 USD = 1,000,000 microdollars)”. The request flow is balance check → execution → usage tracking → atomic balance update. “Model inference is deducted from your balance at the upstream provider's rate with no markup,” and “Kilo is not currently charging a payment-processing fee when you purchase Kilo credits.” A zero balance makes paid-model requests return HTTP 402 with `buyCreditsUrl` metadata. “Paid purchases expire one calendar year after purchase.” Organizations draw on a shared balance with per-user daily limits, auto top-up below a threshold, and minimum-balance alerts.

## Model slots

A Kilo session uses several model slots beyond the main chat model:

| Slot | Purpose |
|---|---|
| main model | The agent’s primary model (per-agent override via frontmatter `model`) |
| `small_model` | Lightweight helper tasks (titles and similar) |
| `subagent_model` | Subagent runs when not overridden |
| `memory_model` | Memory save/recall processing (Kilo/auto/small options) |
| compaction (`agent.compaction.model`) | Model used to summarize during context condensing |
| autocomplete | FIM inline completions — “Autocomplete uses Codestral (`mistralai/codestral-2508`) by Mistral AI, billed through your Kilo account” |

## Auto models (`kilo-auto/*`)

Auto Model is Kilo’s routing layer: instead of a concrete model you select a tier exposed as a virtual `kilo-auto/*` model:

| Tier | Strategy | Cost |
|---|---|---|
| `kilo-auto/frontier` | “routes to top-tier models for planning/architect/debug and high-quality models for coding”; the gateway API exposes tier selection via the `x-kilocode-mode` request header | Paid (highest) |
| `kilo-auto/efficient` | “classifies each request by difficulty and routes to the cheapest benchmark-proven model for that task”; “Efficient is also session-aware: it stays with a model across related turns and only switches when a cheaper option is clearly worth it. If it cannot make a routing decision with confidence, it falls back to a fixed, high-quality baseline model” | Paid (lowest) |
| `kilo-auto/free` | “rotates through available free models” | Free |

Tier assignments are not fixed: “The underlying models behind each tier are updated server-side as better options become available or as providers change pricing.” Auto Free can land on free endpoints with data-handling caveats — NVIDIA free endpoints carry a “Trial use only - do not submit personal or confidential data” warning, logged server-side.

## Custom models

Custom providers/models are declared under `provider.<id>.models` in `kilo.jsonc`, with per-model entries covering `tool_call` support, `reasoning`, context/output `limit`s, `cost`, `modalities`, `variants`, and ID mapping. For catalog metadata, “Kilo ships a snapshot of models.dev and refreshes it hourly. If your model ID matches a known model, catalog values are used as defaults.” A model without a declared context window gets no compaction tracking, and `limit` 0 disables compaction while capping output at 32,000 tokens (overridable via `KILO_EXPERIMENTAL_OUTPUT_TOKEN_MAX`).

The VS Code extension and CLI register deep links of the form `vscode://kilocode.kilo-code/kilocode/switch?model=…&agent=…` to switch model/agent from outside the editor.

## Local models

Local runtimes are first-class providers: **Ollama** (`http://127.0.0.1:11434`), **LM Studio** (`http://127.0.0.1:1234/v1`), and **Atomic Chat** (OpenAI-compatible API on port 1337, with opt-in auto-discovery via `provider.atomic-chat`, `atomicChat.autoDetect`, or an `atomic-chat/...` model ID). The docs caveat that local models “often do not support advanced features such as prompt caching, computer use, and others.”

## Official sources

- [Configuration reference](https://kilo.ai/docs/getting-started/settings/overview) · [Auto Model](https://kilo.ai/docs/code-with-ai/agents/auto-model) · [Custom models](https://kilo.ai/docs/code-with-ai/agents/custom-models) · [Gateway](https://kilo.ai/docs/gateway) · [Gateway authentication](https://kilo.ai/docs/gateway/authentication) · [Usage & billing](https://kilo.ai/docs/gateway/usage-and-billing) · [Models & providers](https://kilo.ai/docs/gateway/models-and-providers) · [BYOK](https://kilo.ai/docs/getting-started/byok) · [Autocomplete](https://kilo.ai/docs/code-with-ai/features/autocomplete) · [Local models](https://kilo.ai/docs/automate/extending/local-models) · [Rate limits & costs](https://kilo.ai/docs/getting-started/rate-limits-and-costs)
