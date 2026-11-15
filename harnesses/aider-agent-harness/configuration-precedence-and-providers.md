# Configuration, precedence, and providers

**Verified:** 2026-11-15

## Four configuration surfaces

Aider options can be set four equivalent ways: a command-line switch, a `.aider.conf.yml` entry, an `AIDER_*` environment variable, or an entry in a `.env` file. The options reference documents an environment-variable name for essentially every switch (for example `--model` ↔ `AIDER_MODEL`), and the `.env` file can carry those same variables.

The documentation does not publish a single cross-source precedence table for these four surfaces. What it does specify precisely is the file-loading behavior below; claims that one source always beats another beyond the documented rules are inferences, not documented behavior.

## YAML config file search order

Aider searches for `.aider.conf.yml` in three places, in this order:

1. Your home directory.
2. The root of your git repo.
3. The current directory.

Existing files are loaded in that order, and files loaded last take priority. Passing `--config <filename>` skips the search and loads exactly one config file.

YAML keys mirror the CLI switch names with underscores converted to dashes. List values can be written as bulleted lists or inline with brackets. The downloadable sample config states that it lists all the valid configuration entries.

## `.env` files and environment variables

Aider loads `.env` files in the same search order — home directory, git repo root, current directory, then any file given with `--env-file` — with later files taking priority. The `.env` file stores API keys and other model settings, and can also set general `AIDER_*` options.

A handful of legacy OpenAI-specific switches (`--openai-api-type`, `--openai-api-version`, `--openai-api-deployment-id`, `--openai-organization-id`) are deprecated in favor of `--set-env ENV_VAR_NAME=value`, which sets an arbitrary environment variable for API settings.

## API keys are not ordinary YAML settings

Key storage is provider-specific:

| Surface | What it accepts |
|---|---|
| `.env` file or shell environment | Keys for every provider (`OPENAI_API_KEY`, `GEMINI_API_KEY`, `OPENROUTER_API_KEY`, `DEEPSEEK_API_KEY`, …) |
| `.aider.conf.yml` | Only `openai-api-key` and `anthropic-api-key` |
| `--api-key provider=<key>` | Sets the corresponding `PROVIDER_API_KEY` environment variable (e.g. `--api-key gemini=xxx` sets `GEMINI_API_KEY=xxx`) |

The YAML restriction is documented in the sample config file itself: only OpenAI and Anthropic keys can appear there. The `.env` file is the documented general-purpose key location.

## Model settings and metadata files

Two project-level files extend aider’s knowledge of models:

| File | Default | Purpose |
|---|---|---|
| `--model-settings-file` | `.aider.model.settings.yml` | Settings for unknown models (edit format, extra params, retries, timeouts) |
| `--model-metadata-file` | `.aider.model.metadata.json` | Context window sizes and costs for unknown models |

`--alias ALIAS:MODEL` defines model aliases, configurable multiple times. `--reasoning-effort` and `--thinking-tokens` set reasoning API parameters; by default aider checks whether a model accepts these settings (`--check-model-accepts-settings`, default True) and warns about unfamiliar models (`--show-model-warnings`, default True).

`--weak-model` names the model used for commit messages and chat-history summarization, and `--editor-model` / `--editor-edit-format` name the model and format used for editor tasks in architect mode. Both default to values derived from the main model.

## Providers go through LiteLLM

“Aider uses the LiteLLM package to connect to LLM providers.” Model names therefore follow LiteLLM conventions — `provider/model` — and `--list-models <partial-name>` returns matching known models. Documented provider pages include OpenAI, Anthropic, Gemini, GROQ, LM Studio, xAI, Azure, Cohere, DeepSeek, Ollama, OpenRouter, GitHub Copilot, Vertex AI, and Amazon Bedrock, plus generic OpenAI-compatible APIs and other LiteLLM-supported providers. Local models are supported through Ollama, LM Studio, or any OpenAI-compatible endpoint.

Model capability is operational, not cosmetic: models weak at returning code edits may fail to produce usable changes, and the docs note that models weaker than GPT-3.5 may not work well. The docs’ “works best” list names specific models, which changes with releases; treat it as a current recommendation, not a fixed support matrix.

Deprecated model shortcut flags (`--4o`, `--sonnet`, `--haiku`, `--deepseek`, `--o1-preview`, and similar) remain but are marked deprecated in favor of `--model`.

## Official sources

- [Options reference](https://aider.chat/docs/config/options.html)
- [YAML config file](https://aider.chat/docs/config/aider_conf.html)
- [Config with .env](https://aider.chat/docs/config/dotenv.html)
- [API key configuration](https://aider.chat/docs/config/api-keys.html)
- [Advanced model settings](https://aider.chat/docs/config/adv-model-settings.html)
- [Model aliases](https://aider.chat/docs/config/model-aliases.html)
- [Reasoning models](https://aider.chat/docs/config/reasoning.html)
- [Other LLMs (LiteLLM)](https://aider.chat/docs/llms/other.html)
- [Model warnings](https://aider.chat/docs/llms/warnings.html)
