# Installation, providers, and profiles

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site main — product sunset May 15, 2026)

> **Status:** Roo Code is sunset. “All Roo Code products will be shut down on May 15, 2026. This includes the Roo Code VS Code Extension, Roo Code Cloud, and Roo Code Router.” The extension repository was archived; the team moved to roomote.dev and recommended Cline; the community forked ZooCode. This page documents the final shipped state (v3.54.0, marketplace ID `RooVeterinaryInc.roo-cline`).

## Install and editors

“Roo Code is a VS Code extension that brings AI-powered coding assistance directly to your editor.” Install methods: “**VS Code Marketplace (Recommended)** - fastest method for standard VS Code and Cursor users”; “**Open VSX Registry** - for VS Code-compatible editors like VSCodium”; and manual VSIX. The install guide covers “VS Code, Cursor, VSCodium, and other compatible editors.” After install, “find the Roo Code icon … in the Activity Bar to open the Roo Code panel.” The product is Apache-2.0 (repo `RooCodeInc/Roo-Code`).

## Connecting a provider

“Roo Code needs an inference provider to access the LLM models that make it work.” First-run guidance recommends Claude Sonnet 4.5 and offers three starts: “**Roo Code Router (Recommended):** the easiest way to access multiple providers tested with Roo Code. No API needed”; **OpenRouter** (“access to multiple AI models from different labs through a single API key”); or **Anthropic** direct. Configuration happens in the welcome screen or Settings → Providers: “If you choose the Roo Code Router, just connect your account and choose `anthropic/claude-sonnet-4-5` from the dropdown.”

Model-choice advice: “Different models vary in how they follow tool instructions, parse formats, and maintain context through multi-step operations, so it may be better to try them later. If you do experiment with other models, choose ones specifically designed for structured reasoning and tool use.”

## Provider landscape

“Roo on the other hand is **model-agnostic**, allowing you to choose the model that best fits your needs.” The provider comparison lists extension-supported providers — Anthropic, OpenAI, OpenRouter, Gemini, AWS Bedrock, Vertex, DeepSeek, Mistral, Moonshot, MiniMax, xAI, Z AI, Requesty, Fireworks, Baseten, SambaNova, LiteLLM, Vercel AI Gateway, VS Code LM API, Ollama, LM Studio, OpenAI-compatible, Qwen Code, and the Roo Code Router — with some providers also available to Cloud Agents. Local/offline options: “**Ollama:** A popular open-source tool for running large language models locally” and **LM Studio**, with the trade-offs documented (“Limited Features: Local models … often do not support advanced features such as prompt caching, computer use, and others”).

**Roo Code Router** (sunset with the suite): “it only needs your Roo Code Cloud account and offers a selection of models we tested and approved to work with Roo Code.” “We never use your data for training and don't keep logs of the prompts themselves,” with the caveat that “task sync or Cloud Agents” require keeping task copies and that free stealth models “will most likely involve the use of your prompts as training data.”

## API configuration profiles

“API Configuration Profiles allow you to create and switch between different sets of AI settings.” Each profile can carry its own provider, keys, model, temperature, thinking budgets, provider-specific settings, diff-edit configuration, and “Rate limit settings” — “Rate limits, which default to 0 (disabled) and typically don't need adjustment, are now configured per profile.” Profiles are managed under Settings → Providers with a `+` button, and modes can pin different profiles.

Cost handling: “Roo Code automatically calculates the estimated cost of each API request based on the configured model's pricing,” and “For reasoning-capable models … Roo Code now includes both normal tokens **and** reasoning / "thought" tokens in its estimates when the provider reports them.” The calc “is an *estimate*.”

## Settings import/export

Settings export “includes your configured API Provider Profiles and Global Settings” — “Crucially, this includes **API keys in plaintext**. Treat this file as highly sensitive.” Import “merges” configurations (“It does **not** delete configurations present in your current setup but missing from the imported file”) and “fails only if **all** profiles are invalid.” An “Automatic Configuration Import” can sync settings from a path on every VS Code start (“If the file isn't found or contains errors, Roo Code starts up normally without blocking your workflow”).

## Official sources

- [Installing](https://roocodeinc.github.io/Roo-Code/getting-started/installing) · [Connecting API Provider](https://roocodeinc.github.io/Roo-Code/getting-started/connecting-api-provider) · [Providers](https://roocodeinc.github.io/Roo-Code/providers/) · [API Configuration Profiles](https://roocodeinc.github.io/Roo-Code/features/api-configuration-profiles) · [Settings Management](https://roocodeinc.github.io/Roo-Code/features/settings-management) · [Local Models](https://roocodeinc.github.io/Roo-Code/advanced-usage/local-models) · [Rate Limits and Costs](https://roocodeinc.github.io/Roo-Code/advanced-usage/rate-limits-costs) · [Roo Code Router](https://roocodeinc.github.io/Roo-Code/roo-code-router/overview)
