# Aider agent harness

**Verified:** 2026-11-15 against the current `Aider-AI/aider` repository (latest tagged release v0.86.0), official documentation at aider.chat, and the release history on `main`  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** aider CLI, YAML/.env configuration, model settings and providers, files and repo map, git integration, chat modes and edit formats, in-chat commands, watch files, lint/test automation, headless scripting, Docker and browser surfaces, analytics and privacy  
**Provenance:** Official aider sources unless explicitly marked otherwise

Aider is AI pair programming in your terminal. It is git-centric — edits are committed as they are made, with configurable attribution — and its context model is explicit: files are added to the chat, and a token-budgeted repository map summarizes the rest. Aider documents no MCP support, no subagents, no skills, and no lifecycle hooks; its automation surface is lint/test commands, watch-files, command files, and CLI scripting.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, precedence, and providers](configuration-precedence-and-providers.md) | CLI switches, YAML config, environment variables, `.env`, key storage, model settings/metadata files, aliases, and LiteLLM providers |
| [Instructions, context, and git](instructions-context-and-git.md) | Files in the chat, conventions and read-only files, the repository map, ignore files, auto-commits, undo, commit messages, and attribution |
| [Commands, modes, and edit formats](commands-modes-and-edit-formats.md) | Chat modes, architect/editor split, edit formats, in-chat commands, watch files, editor, voice, web, and notifications |
| [Automation, scripting, and security](automation-scripting-and-security.md) | Lint and test integration, headless runs, command files, Python API, Docker, install/upgrade lifecycle, analytics, and privacy |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Aider supports MCP servers.”** No MCP integration appears anywhere in aider’s official documentation or README; its extension points are lint/test commands, `/run`, watch files, and an explicitly unsupported Python API.
- ❌ **“Aider has subagents or agent teams.”** No subagent, agent-team, or orchestrator concept is documented; aider runs one model conversation against the files in the chat.
- ❌ **“CONVENTIONS.md is picked up automatically.”** Conventions load only through `--read`, in-chat `/read-only`, or a `read:` entry in `.aider.conf.yml`.
- ❌ **“Aider sends your entire repository to the LLM.”** The repo map is a ranked summary under a token budget (1k tokens by default), not the tree.
- ❌ **“Aider never commits without asking.”** In a git repo, auto-commits and dirty-commits are enabled by default; aider commits its edits and preexisting dirty changes.
- ❌ **“Any provider’s API key can go in `.aider.conf.yml`.**” Only OpenAI and Anthropic keys are accepted in the YAML config; all providers’ keys belong in the `.env` file or environment.
- ❌ **“Aider ships a desktop app or an IDE extension.”** Aider is a terminal tool with an experimental browser UI (`--gui`/`--browser`); the documented IDE story is `--watch-files`, not a plugin.
- ⚠️ **“Auto-lint and auto-test both run after every edit by default.”** Auto-lint defaults on; auto-test defaults off and needs `--test-cmd` plus `--auto-test`.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| YAML config search order | Home dir → git root → current dir; loaded in that order, last file wins; `--config` loads exactly one file |
| `.env` search order | Home dir → git root → current dir → `--env-file` path; loaded in that order, last file wins |
| Option env-var mapping | Every option has an `AIDER_<UPPER_SNAKE>` equivalent (e.g. `AIDER_MODEL`) |
| Model settings file | `.aider.model.settings.yml` (settings for unknown models) |
| Model metadata file | `.aider.model.metadata.json` (context window and costs for unknown models) |
| Ignore file | `.aiderignore` in the git root by default; gitignore-style syntax |
| Provider layer | LiteLLM; model names use `provider/model` form, discovered with `--list-models` |
| Keys | All providers in `.env`; only OpenAI/Anthropic accepted in YAML; `--api-key provider=<key>` sets `PROVIDER_API_KEY` |
| Auto-commits / dirty-commits | On by default in a git repo |
| Commit attribution | `(aider)` in author/committer names, or `Co-authored-by` trailer; both default on, co-authored-by takes precedence unless the others are explicitly set |
| Repo map budget | `--map-tokens` defaults to 1k; `0` disables |
| Default permission prompts | Confirmations for suggested shell commands and similar actions; `--yes-always` answers yes to every confirmation |
| History files | `.aider.chat.history.md`, `.aider.input.history` by default; `--llm-history-file` optional |
| Headless run | `aider --message "…" [--yes-always]` processes one message and exits |
| Browser UI | `aider --gui` (alias `--browser`); experimental |
| Analytics | Opt-in; PostHog; never code, prompts, chats, keys, or personal info; `--analytics-disable` opts out permanently |

## Official sources

- [Project repository](https://github.com/Aider-AI/aider)
- [Configuration](https://aider.chat/docs/config.html)
- [Options reference](https://aider.chat/docs/config/options.html)
- [YAML config file](https://aider.chat/docs/config/aider_conf.html)
- [Config with .env](https://aider.chat/docs/config/dotenv.html)
- [API key configuration](https://aider.chat/docs/config/api-keys.html)
- [Advanced model settings](https://aider.chat/docs/config/adv-model-settings.html)
- [Connecting to LLMs / other providers](https://aider.chat/docs/llms/other.html)
- [In-chat commands](https://aider.chat/docs/usage/commands.html)
- [Chat modes](https://aider.chat/docs/usage/modes.html)
- [Edit formats](https://aider.chat/docs/more/edit-formats.html)
- [Specifying coding conventions](https://aider.chat/docs/usage/conventions.html)
- [Repository map](https://aider.chat/docs/repomap.html)
- [Git integration](https://aider.chat/docs/git.html)
- [Linting and testing](https://aider.chat/docs/usage/lint-test.html)
- [Watch files](https://aider.chat/docs/usage/watch.html)
- [Scripting aider](https://aider.chat/docs/scripting.html)
- [Analytics](https://aider.chat/docs/more/analytics.html)
- [Privacy policy](https://aider.chat/docs/legal/privacy.html)
- [Docker](https://aider.chat/docs/install/docker.html)
- [Release history](https://aider.chat/HISTORY.html)
