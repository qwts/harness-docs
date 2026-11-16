# Models, providers, and reasoning

**Verified:** 2026-11-15

## Default model

“Mirror uses Reflection and `Beam-1.0-501B-A23B` unless you choose otherwise.” The API model record for Beam (the model behind the `reflection` provider) lists a 1,048,576-token context window, 131,072 max output tokens, a 32768 `default_parameters.max_tokens`, knowledge cutoff June 30 2026, and `"mandatory": true` reasoning with `"default_effort": "max"`; the compatibility guide notes the API “accepts text input only,” and the quickstart's Hermes section confirms image requests “fail with `400 invalid_value`.” The CLI caps generated tokens per request at 64000 (`--max-tokens` “Defaults to 64000”).

## Switching models

“Run `/model` to open the picker. It lists Reflection, the other supported providers, and any models you've added to `models.json`.” You can also name the target directly (`/model Beam-1.0-501B-A23B`, `/model reflection`). Scope: “The choice applies to the current workspace only. Mirror remembers your choice, so the workspace uses it the next time you open it, and new workspaces start with the model of the workspace you created them from. Run `/status` to check which model is active.”

Without flags, launch resolution is: “1. The model you last used in this workspace. 2. The model you last selected anywhere. 3. The first entry in `models.json`. 4. Reflection and `Beam-1.0-501B-A23B`.” `--model` and `--api` “apply to one launch only, while `/model` changes the saved selection.”

## Reasoning effort

“Run `/effort` to choose a level, or pass one directly” (`/effort high`). “The levels are `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`. Mirror starts at `max` unless the workspace has a saved effort or you pass `--reasoning-effort`. **Provider default** (`none`) sends no effort setting at all and leaves the choice to the provider. Models don't all support every level.” “Like the model, each workspace remembers its reasoning effort.” Gemini is the documented exception: “Choose **Provider default**, `low`, `medium`, or `high` reasoning effort; the other effort levels aren't supported. If your workspace uses another level, run `/effort medium` before `/model gemini`.”

## Providers

“Mirror works with several model APIs. Each needs its own API key, read from an environment variable. Exporting a key doesn't select the provider; choose it with `/model` or `--api`.”

| Provider | `--api` value | API key |
|---|---|---|
| Reflection | `reflection` | `REFLECTION_API_KEY`, or the key saved when you first connected |
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` |
| OpenAI | `openai-responses` | `OPENAI_API_KEY` |
| Anthropic | `anthropic-messages` | `ANTHROPIC_API_KEY` |
| Google Gemini Developer API | `gemini` | `GEMINI_API_KEY` with paid API access |
| ChatGPT plan | `openai-codex-responses` | A Codex sign-in. Run `codex login --device-auth` first. |
| Any OpenAI-compatible Chat Completions endpoint | `openai-chat-completions` | `OPENROUTER_API_KEY`, if the endpoint needs one |

“Passing `--api` without `--model` selects that provider's default model. You can pass a key with `--api-key` instead of an environment variable, but it then appears in your shell history.” For ChatGPT plan access “Mirror reads `~/.codex/auth.json`. Set `MIRROR_CODEX_AUTH_PATH` to use another credential file. For a Chat Completions endpoint that can't stream, `--api openai-chat-completions-non-streaming` is also accepted.”

Gemini specifics: “The Gemini provider supports `gemini-4-argon` and requires paid Developer API access. A Google AI subscription doesn't supply API credentials.” It uses the Developer API endpoint — “For Vertex AI or a custom endpoint, use `openai-chat-completions` with the endpoint's model and credentials.”

## Reflection sign-in

“`/login` checks whether Mirror has a Reflection API key and opens the **Connect to Reflection** prompt if it doesn't. `/logout` deletes the key saved in `~/.mirror/reflection_api_key`. … Mirror saves a key you enter without checking it, so use `/logout` to replace a mistyped one. `/logout` doesn't affect `REFLECTION_API_KEY`, which takes precedence over the saved key whenever it's set.” The quickstart adds: the saved key is “readable only by your user,” the input is masked, and “Like other local credentials, the saved key can be read by any command that runs as your user.”

## Local and custom endpoints

“Point Mirror at any server that implements OpenAI Chat Completions, such as a local inference server, with `--base-url` and the model ID the server expects.” “No API key is needed if the endpoint doesn't require one. If it does, set `OPENROUTER_API_KEY` or pass `--api-key`.” Endpoints can also come from the environment: `OPENROUTER_BASE_URL` for Chat Completions, `OPENAI_BASE_URL` for OpenAI, `ANTHROPIC_BASE_URL` for Anthropic. One trap: “If `OPENROUTER_BASE_URL` is set and you don't pass `--api`, Mirror uses Chat Completions with that endpoint instead of Reflection. Unset it, or pass `--api reflection`, to use Reflection.” And “The `reflection` provider always uses Reflection's endpoint and doesn't accept another base URL.”

## models.json

“To keep models you use often in the `/model` picker, list them in `~/.mirror/models.json`. Each entry needs an `id` (the model ID sent to the provider) and a `label` (the name shown in the picker), and can set a `description`, an `api`, and a `base_url`. … Selecting an entry applies its provider and endpoint along with the model, and `--api` or `--base-url` on the command line override them. Keep API keys in environment variables, not in this file. To use a different file, set `MIRROR_MODELS_CONFIG` to its path.”

## Official sources

- [Mirror models and providers](https://developers.reflection.ai/mirror-models.md) · [Models (API reference)](https://developers.reflection.ai/models.md) · [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md) · [Coding-agents quickstart](https://developers.reflection.ai/coding-agents-quickstart.md)
