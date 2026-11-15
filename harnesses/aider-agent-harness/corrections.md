# Aider corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| Aider supports MCP servers. | ❌ FALSE | No MCP integration appears anywhere in aider’s official documentation or README. Its automation points are lint/test commands, `/run`, watch files, command files, and CLI scripting flags. |
| Aider has subagents or agent teams. | ❌ FALSE | No subagent, agent-team, or orchestrator concept is documented; aider runs one model conversation against the files in the chat. |
| Aider ships a desktop app. | ❌ FALSE | Aider is a terminal tool; the only other documented surface is the experimental browser UI launched with `--gui` (alias `--browser`). |
| Aider has an official IDE extension. | ❌ FALSE | The documented IDE story is `--watch-files`: save AI comments in any editor and aider responds in its terminal session. No extension is documented. |
| `CONVENTIONS.md` is discovered and loaded automatically. | ❌ FALSE | Conventions load only via `aider --read CONVENTIONS.md`, in-chat `/read-only`, or a `read:` entry in `.aider.conf.yml`. |
| The in-chat command for adding reference-only files is `/read`. | ❌ FALSE | The current CLI implements `/read-only`; there is no `/read` command handler. The conventions guide’s `/read CONVENTIONS.md` spelling is outdated — use `aider --read` at launch or `/read-only` in-chat. |
| Aider sends the entire repository to the LLM. | ❌ FALSE | The repo map is a ranked summary of important classes, functions, types, and signatures fitted to a token budget that defaults to 1k tokens. |
| The repo map includes every symbol of every file. | ❌ FALSE | A graph ranking algorithm selects the most important definitions that fit the active budget; only the most-referenced identifiers appear. |
| `.aiderignore` replaces `.gitignore` as the only ignore mechanism. | ❌ FALSE | Both apply: `.aiderignore` (gitignore-style, git root by default) removes paths from consideration, while files listed in `.gitignore` are excluded from aider’s editing scope unless `--add-gitignore-files` is enabled. Aider also offers to add `.aider*` entries to `.gitignore` by default. |
| Aider never commits anything without asking. | ❌ FALSE | In a git repo, `--auto-commits` and `--dirty-commits` default to True: aider commits its edits and commits preexisting dirty changes before editing them. |
| `/undo` reverts your own recent changes too. | ❌ FALSE | `/undo` undoes the last git commit if it was done by aider; your commits are untouched. |
| Any provider’s API key can go in `.aider.conf.yml`. | ❌ FALSE | Only `openai-api-key` and `anthropic-api-key` are accepted in the YAML config; the `.env` file or environment handles keys for every provider. |
| Environment variables always outrank `.aider.conf.yml`. | ❓ UNVERIFIED | The docs document four equivalent ways to set an option and precise per-file load order (home → git root → cwd, last file wins) for both config file types, but no single cross-source precedence table. Treat source-ranking claims beyond that as unverified. |
| Auto-lint and auto-test both run after every edit by default. | ⚠️ PARTIAL | Auto-lint defaults to True, but auto-test defaults to False and additionally needs `--test-cmd` to know what to run. |
| Aider only works with OpenAI and Anthropic models. | ❌ FALSE | Aider uses LiteLLM to connect to providers; OpenAI, Anthropic, Gemini, GROQ, LM Studio, xAI, Azure, Cohere, DeepSeek, Ollama, OpenRouter, GitHub Copilot, Vertex AI, Amazon Bedrock, OpenAI-compatible APIs, and local models are documented. |
| Formatters can be used directly as `--lint-cmd` values. | ⚠️ PARTIAL | They run, but formatters that modify files and exit non-zero make aider think there is a lint error; the docs recommend a wrapper script that runs the formatter twice. |
| `/test <command>` forwards extra arguments to the test command. | ❌ FALSE | Aider runs the test command without any arguments and shares output when it exits non-zero. |
| Aider works without git with no loss of functionality. | ⚠️ PARTIAL | `--no-git` exists, but the docs recommend against it: there are no aider commits to undo, and the docs say to keep sensible backups. |
| Aider writes commit messages with the main chat model. | ❌ FALSE | The `--weak-model` receives a copy of the diffs and chat history and produces the message, following Conventional Commits by default. |
| Aider always appends “(aider)” to the git author name. | ❌ FALSE | By default aider appends a `Co-authored-by` trailer; the “(aider)” name suffixes apply under `--attribute-author`/`--attribute-committer` (default True) unless co-authored-by takes precedence, and co-authored-by is disabled when either is explicitly set to True. |
| Aider runs pre-commit hooks on its commits by default. | ❌ FALSE | `--git-commit-verify` defaults to False, so aider commits with `--no-verify`, skipping pre-commit hooks; enabling it runs the hooks. |
| Analytics are on by default and include code. | ❌ FALSE | Analytics are opt-in only, tied to a random UUID4 identifier, and the docs state aider never collects code, prompts, chats, keys, or personal info. |
| `--analytics-disable` turns analytics off for the current session only. | ❌ FALSE | It opts out permanently; answering “no” to the opt-in prompt also disables analytics permanently. Session-level control uses `--analytics`/`--no-analytics`. |
| Aider provides a stable, supported Python SDK. | ❌ FALSE | The Python API “is not officially supported or documented, and could change in future releases without providing backwards compatibility.” |
| Aider offers ACP, headless JSON, or RPC integration surfaces. | ❌ FALSE | Headless operation is `--message`/`--message-file` (process one message, then exit); no ACP, JSON, or RPC surface is documented. |
| The Docker image behaves identically to a local install. | ⚠️ PARTIAL | In Docker, `/run` commands execute inside the container, `/voice` needs host audio access, local git identity must be configured, and the core image reinstalls optional extras on each launch. |
| `aider --gui` opens a stable, supported web app. | ⚠️ PARTIAL | It launches aider’s documented but experimental browser UI, which edits the local git repo like the terminal version. |
| The edit format setting is cosmetic. | ❌ FALSE | Format choice is operational: `whole` returns entire files, `diff`-style formats return search/replace blocks, `diff-fenced` and `udiff` exist for specific model families’ behaviors, and the optimal format is configured per model. |
| Architect mode uses one model. | ❌ FALSE | An architect model proposes changes and a separate editor model translates them into file edits, costing two LLM requests per change. |
| Deprecated model shortcuts like `--4o` or `--sonnet` are the current way to select models. | ❌ FALSE | They remain only for backward compatibility and are marked deprecated; use `--model`. |
| Watch files require a specific IDE plugin. | ❌ FALSE | `--watch-files` watches the repo directly; any editor that saves files works, using `AI`/`AI!`/`AI?` comments in `#`, `//`, or `--` styles. |
| `AI!` comments make aider execute the commented commands. | ❌ FALSE | They are prompts: `AI!` triggers aider to make changes, `AI?` triggers an answer, and aider asks the model to remove the AI comments as part of the change. |
| Aider updates itself silently in the background. | ❌ FALSE | `--check-update` (default True) only checks on launch; upgrading requires `--upgrade` or `--install-main-branch`, and release notes are shown per `--show-release-notes` (ask by default). |
| The `whole` edit format is always worse than `diff`. | 💬 OPINION | The docs describe tradeoffs — `whole` is simple but slow and costly, `diff` is efficient — and configure the optimal format per model rather than ranking them universally. |
