# Configuration, providers, and models

**Verified:** 2026-11-15

## Configuration layers and data directory

Four layers: current-run CLI options (override one process), runtime config `<data-dir>/config.yaml` (“Stores default model, permission mode, and MiniMax source”), provider credentials via `mcode provider ...`, and project rules (`AGENTS.md` in the workspace). “Command-line overrides apply only to the current run and do not silently rewrite global configuration.” The default data root is `~/.minimax` — it “stores configuration, sessions, logs, plugins, skills, and other runtime data” — relocated with `MINIMAX_DATA_DIR`; “`MAVIS_DATA_DIR` is a compatibility fallback; `MINIMAX_DATA_DIR` wins when both are set.”

`config.yaml` runtime fields: `defaultModel` (`provider/model`, may carry `#variant`), `defaultModelVariant`, `permissionMode` (`default`, `auto`, `bypassPermissions`, `off`), and `minimaxModelSource` (`token_plan`, `minimax_api_key`). Model references use `provider/model#variant`; `/model [filter]` searches in the TUI and `--model` is a one-run override. New in 0.7.x: beta flags under `beta` (`ptcScripts`, `ptcStore`) enable the PTC `run_code` scripting features, off by default in public production builds.

## Sign-in and MiniMax credentials

“mcode login” signs in a mainland-China account; `mcode login --region global` a Global one (`--region` accepts `cn` or `global`; ACP has `mcode acp login --region <region>`). Sign-in completes in the browser via a temporary loopback callback service; `mcode logout` “clears the local sign-in state, prints a complete browser sign-out link,” and on headless hosts you copy the link yourself. “Login callback URLs can contain temporary credentials and must not be shared or committed.”

API keys come from environment variables, not config files: “The default variable is `MCODE_PROVIDER_API_KEY`,” then `mcode provider set-minimax-key` saves and selects it; `--api-key-env <name>` picks another variable, and “The CLI reads the key but does not print it.” `mcode provider list` shows “only managed-login or masked-key status.” Proxy support: `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, `NO_PROXY` (plus lowercase variants), and “The CLI always bypasses `localhost`, `127.0.0.1`, and `::1` for login callbacks and local services.”

## Custom providers

`mcode provider add --name ... --base-url ... --api-format ... --model ... --use` (at least one `--model`, repeatable; `--use` selects the first as default). “Supported API formats are `anthropic-messages`, `openai-completions`, `openai-responses`, and `google-generative-ai`” — for Gemini, “a Base URL with only the host gets `/v1beta` appended automatically.” Management: `provider list [--json]`, `provider test <id> [--model | --json]`, `provider remove <id> --yes`; “The MiniMax OAuth provider is managed with `mcode login` or `/login`, rather than `provider test`.”

## Context capacity and status line

New sessions inherit the model's context capacity. From 0.4.12 the default status line “shows the known total capacity, such as `Context 1M` or `Context 512K`, until valid usage is available. It then switches to the remaining percentage, such as `Context 80% left`, without showing both indicators” — `/statusline` items `context-window` and `context-remaining` can both be enabled explicitly and “do not follow the default automatic switching rule.” When capacity is unavailable, “the capacity item is hidden instead of displaying another session's value.” `/status` shows “version, account, model, runtime configuration, and quota summaries” including “available 5-hour and weekly quota summaries with remaining percentages and reset countdowns.”

## Plans, credits, and BYOK

M Plan is the subscription (“builds on the capabilities of Token Plan”; “Subscription prices and text usage quotas stay the same,” select tiers get the full model lineup incl. H3). Daily check-in earns 400 points (1,000 on days 4 and 7; a full week 4,000), valid 30 days from crediting, spendable across task scenarios. The desktop account settings show plan status, validity, credit balance, quotas, and billing entries. Desktop BYOK (Settings → Usage & model → Custom model): preset providers supply base URL/format/models, or pick Others and enter your own; `Fetch` retrieves the model list without saving, the per-model connectivity test checks response, and Save “does not automatically test the connection or switch the model used by the current task.” Quota displays include “the 5-hour limit, weekly limit, and video limit” (WebUI wording).

## Official sources

- [Configuration and models](https://agent.minimax.io/docs/cli/configuration.md) · [Features and configuration](https://agent.minimax.io/docs/cli/features.md) · [Security and permissions](https://agent.minimax.io/docs/cli/security.md) · [M Plan and Credits](https://agent.minimax.io/docs/code/account/usage.md) · [Custom Models / BYOK](https://agent.minimax.io/docs/code/account/byok.md) · [Changelog](https://agent.minimax.io/docs/changelog.md)
