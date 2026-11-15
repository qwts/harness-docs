# Automation, scripting, and security

**Verified:** 2026-11-15

## Lint and test integration

Aider ships built-in linters for most popular languages and lints files it edits by default (`--auto-lint`, default True; `/lint` for manual runs). Custom linters are configured per language with `--lint-cmd "language: cmd"` (repeatable). The command contract: it receives the filenames to lint, prints errors on stdout/stderr, and exits non-zero when there are problems. Code formatters that rewrite files and exit non-zero confuse this contract — the documented workaround is a wrapper script that runs the formatter twice, so the second pass exits 0 unless a real problem remains.

Tests run only where configured:

- `/test <command>` runs the command without arguments and shares output with aider when it exits non-zero, so aider can fix the failures.
- `--test-cmd <command>` plus `--auto-test` (default False) runs the suite after each AI edit.
- For compiled languages the docs suggest `--lint-cmd` for per-file compile checks and a build-and-test `--test-cmd` (e.g. `dotnet build && dotnet test`).

## Headless and scripted runs

`--message "…"` (aliases `--msg`, `-m`) sends one message, applies the edits, and exits; `--message-file` does the same from a file. Both disable chat mode. `--yes-always` answers yes to every confirmation; `--dry-run` skips file modifications. One-shot variants include `--commit` (commit pending changes with a generated message, then exit), `--apply FILE` (apply edits from a file, a debug aid), and `--apply-clipboard-edits`. `--exit`, `--show-repo-map`, and `--show-prompts` are debug surfaces that run startup activities and print state.

Shell loops over files are the documented batching pattern. `/save` and `/load` (or `--load`) execute command files at launch.

The Python API (`Coder.create(...)` then `coder.run(...)`) exists but the docs are explicit: it “is not officially supported or documented, and could change in future releases without providing backwards compatibility.” Aider documents no ACP server, no JSON-RPC surface, and no SDK; headless integration is the CLI flags above.

## No MCP, subagents, skills, or hooks

Aider’s official documentation and README describe no MCP client or server support, no subagent or agent-team model, no skills, and no lifecycle hooks. The automation surface is exactly: lint/test commands, `/run`, watch files, command files, and the CLI scripting flags. Documentation that attributes MCP servers, hooks, or agent teams to aider describes some other tool.

## Install, upgrade, and release channels

- The `aider-install` installer (Python 3.8–3.13) installs aider in its own separate Python environment; uv-based one-liners (`curl -LsSf https://aider.chat/install.sh | sh`) install aider plus Python 3.12 if needed.
- `uv tool install --force --python python3.12 --with pip aider-chat@latest`, `pipx install aider-chat` (Python 3.9–3.12), and plain `pip` (less recommended, venv advised) are documented alternatives; system package managers are discouraged for dependency reasons.
- `aider --upgrade` upgrades from PyPI; `--check-update` (default True) checks on launch; `--just-check-update` returns the status in the exit code; `--install-main-branch` installs from `main`; `--show-release-notes` asks about notes on first run of a new version.

## Docker surface

Official images are `paulgauthier/aider` (core) and `paulgauthier/aider-full` (all optional extras). The full image includes interactive help, the browser GUI, and Playwright out of the box; the core image installs extras on first use, and because containers are ephemeral, core-image extras are reinstalled on each launch. The documented run mounts the current directory (`--volume $(pwd):/app`), launches from the repo root, and needs local `git config user.name`/`user.email` because the container lacks your global git config. `/run` commands execute inside the container, and `/voice` needs host audio access.

## Analytics and privacy

Analytics are opt-in: “Analytics are only collected if you agree and opt-in.” By default aider enables the prompt for a random subset of users (`--analytics` defaults to `random`). Collected events cover LLM usage and token counts, edit formats, feature and command frequency, and exceptions/errors, associated with a random UUID4 identifier — the docs state aider “never collects your code, prompts, chats, keys or any personal info.” Data goes to PostHog (app.posthog.com by default; self-hosting via `--analytics-posthog-host` and `--analytics-posthog-project-api-key`). `--analytics-log filename.jsonl` writes a full local event log, which can be combined with `--no-analytics` to log locally without reporting. `--analytics-disable`, or answering “no” to the opt-in prompt, disables analytics permanently.

The privacy policy separately covers the aider.chat website (cookies, IP address, device identifiers) and confirms analytics are not tied to directly identifiable user information; tools are hosted in the United States. It also documents the opt-out path: “You can turn off analytics collection when using our programming tools.”

## Official sources

- [Linting and testing](https://aider.chat/docs/usage/lint-test.html)
- [Scripting aider](https://aider.chat/docs/scripting.html)
- [Options reference](https://aider.chat/docs/config/options.html)
- [Installation](https://aider.chat/docs/install.html)
- [Aider with docker](https://aider.chat/docs/install/docker.html)
- [Analytics](https://aider.chat/docs/more/analytics.html)
- [Privacy policy](https://aider.chat/docs/legal/privacy.html)
- [Release history](https://aider.chat/HISTORY.html)
