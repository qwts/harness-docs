# Installation, authentication, and models

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site `docs.mistral.ai/vibe` main)

## Product identity and surfaces

“Vibe is Mistral's **unified agent** for productivity and coding tasks … available across web, mobile, your code editor, and your terminal,” running in two modes: **Work** (the web/mobile productivity experience) and **Code** — “development in your terminal, in VS Code, or as remote cloud sessions.” This set documents **Vibe Code**, the coding mode (the issue's scope: Mistral Vibe CLI/coding mode, not generic vibe coding or the Mistral model APIs).

“**One agent, three surfaces.** Vibe Code runs as the CLI, the VS Code extension, and Vibe Code Web. They share the same engine, configuration, and customization layer (agents, skills, MCP servers).” A fourth path: “Vibe also implements the Agent Client Protocol (ACP) and is published in the ACP registry, so you can run it in other ACP-compatible clients, including JetBrains IDEs.” Note: “**Chat and Work are now one.**” (merged September 22); the Mistral Code Enterprise JetBrains plugin “It's deprecated”; the VS Code extension has a documented “Migration from Mistral Code Enterprise” page.

Version anchors: the security advisory fixes land in “Mistral Vibe **2.25.4**, released on **12 September 2026**”; admin config requires Vibe Code CLI `2.24.2` and VS Code extension `1.16`. Release notes ship at docs.mistral.ai/resources/release-notes (latest entries 2026-11-05).

## Install

- **macOS / Linux** — one-line installer: `curl -LsSf https://mistral.ai/vibe/install.sh | bash`. The installer “Checks whether `uv` is installed and installs it if needed”, “Installs or upgrades `mistral-vibe` with `uv tool install` or `uv tool upgrade`”, and “Makes the `vibe` and `vibe-acp` commands available, provided your `PATH` is configured.”
- **Manual** — `uv tool install mistral-vibe`; for voice mode on platforms without a prebuilt audio library: `uv tool install "mistral-vibe[voice]"`.
- **Windows** — “Vibe runs on Windows, but macOS and Linux are the primary supported platforms”: install `uv` first, then `uv tool install mistral-vibe`.

Start with `vibe` in the project root, or with an initial prompt: `vibe "Refactor the main function in cli/main.py to be more modular."` First run starts a setup flow; rerun anytime with `vibe --setup`. Point the CLI at another project with `--workdir /path/to/project`.

## Authentication

“Vibe Code CLI needs a **Mistral API key** to call hosted models” — but “**Browser-based sign-in is enabled by default** when your config targets a model with a Mistral provider. The flow provisions and stores credentials for you, so no API key is needed for typical Mistral-provider setups.”

When a key is used, “the CLI uses the first key it finds, in this order: your shell environment, then `~/.vibe/.env`, then your OS keyring (for example, macOS Keychain or Windows Credential Manager), where the setup flow saves keys.” The shell wins: “Environment variables take precedence over values stored in `~/.vibe/.env` and in the keyring.” `MISTRAL_API_KEY` is the variable. “The `.env` file is for **credentials only**. General CLI configuration belongs in `config.toml`.”

Introspection: “Run `/status` in a session to see the active model and provider, which credential source is in use, and which sources it overrides. Run `/whoami` to see your signed-in account, Workspace, plan, API key scope, and the credential source in use.”

## Providers and models

Providers and models are presets in `config.toml` (`[[providers]]`, `[[models]]` with an `alias`); `active_model` selects one, and `allowed_models` is “Allow-list of model name patterns (exact, glob, or `re:` regex), matched against the model names sent to the inference API, not their aliases.” Supporting models: `compaction_model` (“Must use the active model's provider”), `vision_model` (describes images when the active model lacks vision), `auto_compact_threshold`, output caps (`agent_max_output_tokens`, `compaction_max_output_tokens`), and `output_limit_retries` (default `6` — “discards an answer cut off at the output token limit and asks for it again”). `/model` and `/thinking` pick model and thinking level per session or persist them.

**Offline / self-hosted**: “The Vibe Code CLI supports **any model served behind an OpenAI-compatible API**.” Recommended local models: “**Devstral Small 2**: dense 24B, tuned for agentic and code tasks” and “**Mistral Small 4**: 119B mixture-of-experts … 256k context,” with vLLM deployment guidance (FP8 ~128k on H100/A100).

## Corporate certificates

`enable_system_trust_store` (default `false`) “Validates HTTPS certificates against the operating system trust store instead of Vibe's bundled roots,” and `/proxy-setup` configures proxy and SSL settings.

## Official sources

- [Vibe overview](https://docs.mistral.ai/vibe) · [Choose CLI, VS Code, or web](https://docs.mistral.ai/vibe/code/choose-cli-vscode-web-sessions) · [Install and setup](https://docs.mistral.ai/vibe/code/cli/install-setup) · [API keys and profiles](https://docs.mistral.ai/vibe/code/cli/api-keys-profiles) · [Offline models](https://docs.mistral.ai/vibe/code/cli/offline-models) · [Configuration reference](https://docs.mistral.ai/vibe/code/cli/configuration-reference) · [Release notes](https://docs.mistral.ai/resources/release-notes) · [Advisory MAI-2026-003](https://docs.mistral.ai/resources/security-advisories/MAI-2026-003)
