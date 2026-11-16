# Installation, surfaces, and authentication

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main; app changelog 2026.11.05 / v0.2026.11.04.08.17)

## The three agent surfaces

“**You reach the same agent three ways.** Which one you pick depends on where you're working, not on what the agent can do”:

- **In the Warp app** — “The full experience, with code review, agent management, and the terminal beside your conversation.” The app is the desktop product for “macOS (Intel and Apple silicon), Windows (x86\_64 and ARM64), and Linux (x86\_64 and ARM64)” (`brew install --cask warp`, `winget install Warp.Warp`, or the download).
- **In any terminal, with the Warp Agent CLI** — “Run the `warp` command to start a conversation in any terminal emulator, over SSH, or on a machine where the Warp app isn't installed.”
- **In the cloud, as a cloud agent** — “Run the agent in the background from a trigger, a schedule, or an integration, without keeping a session open,” on the Automation Platform (see [sessions, planning, and orchestration](sessions-planning-and-orchestration.md) for the boundary).

The Warp Agent itself is “Warp's built-in coding agent. It works through multi-step tasks on its own, running commands in a real terminal and using the output to decide what to do next. You stay in the loop, approving actions and reviewing diffs before anything lands.”

## Installing the Warp Agent CLI (`warp`)

“The install script is the recommended method, because it's the only one that keeps the CLI up to date on its own”:

- macOS/Linux: `curl -fsSL https://app.warp.dev/download/agent-cli | bash`
- Windows (PowerShell): `Invoke-RestMethod "https://app.warp.dev/download/agent-cli.ps1" | Invoke-Expression`
- Homebrew (macOS 14+ / Linux): `brew install --cask warp-agent-cli` — “Homebrew installations don't update themselves.”

Platforms: “macOS (Apple Silicon or Intel), Linux (x64 or Arm64), or Windows (x64 or Arm64).” Verify with `warp --version`.

Updates: with the install script, “the CLI updates itself. While it runs, it periodically checks for a newer version, downloads it in the background, and stages it without interrupting your session,” taking effect at next launch. Homebrew installs check and notify but require `brew upgrade --cask warp-agent-cli`. Disable checks with `general.autoupdate_enabled = false` in settings, or `WARP_TUI_DISABLE_AUTOUPDATE` for a single launch.

## What the CLI is

“The Warp Agent CLI is a standalone terminal program that runs the Warp Agent, the same agent that powers the Warp app. … The CLI is also a native terminal multiplexer. It creates and maintains its own pseudoterminal (PTY), which lets you and the agent execute commands and interact with running processes through the same terminal, including prompts and full-screen terminal apps.”

It does not require the app: “The CLI doesn't require the Warp app to be installed.” Coming from the app, `/tui-migrate-setup` “copies compatible settings and global MCP server definitions from the app, and asks for approval before changing anything. Credentials and OAuth state are never copied,” while “Rules and skills don't need migration: the CLI and the Warp app both discover them from the same file locations.”

## The legacy Oz CLI (`oz`)

Warp's cloud-management CLI predates the local one: “The Oz CLI (the `oz` binary, which previously shipped as `warp-cli`) is a supported legacy interface. Existing users do not need to change their workflows. … Warp will publish guidance before any support change.” It is “the command-line tool for running and managing Warp's cloud agents from any terminal, script, or CI pipeline,” bundled with the desktop app (install to `PATH` via the Command Palette on macOS) or available standalone. Cloud runs from it look like `oz agent run-cloud --prompt "..."` with no `--harness` flag (“the Warp Agent is the default when no harness is specified”) or `--harness oz`. Scripts should check “the owning reference before changing” which CLI they call.

## Authentication

- **Interactive login** — start `warp`; “The CLI shows a verification link and a device code, and opens the link in your browser. Sign in or create an account, then approve the login.” “You stay signed in across sessions. To sign out later, run `/logout`, which returns the CLI to its login screen and signs out your browser session too.” The start screen lists “the rules, skills, and MCP servers it discovered for your current directory.”
- **Headless/CI** — “Authenticate with a Warp API key instead by setting the `WARP_API_KEY` environment variable”: `WARP_API_KEY=YOUR_API_KEY warp`. `--api-key` takes the same value, but “Prefer supplying the key through the `WARP_API_KEY` environment variable … Command-line arguments can be captured in shell history and process listings. Prefer the `WARP_API_KEY` environment variable, ideally populated from a secret manager.” Keys are created “in the Warp app under **Settings** > **Cloud platform** > **API keys**.”
- **Account status** — `/status` opens “a read-only panel with the installed version, the current conversation's name and ID, the working directory, and the organization and email you're signed in as.” `/usage` shows “your plan, credit balances and consumption, and links to manage billing or upgrade”; `/upgrade` and `/manage-billing` open the respective pages; `/team` switches teams — “The CLI re-scopes the current session in place, unlike the Warp app, which opens a new window for the team you pick.”

## Official sources

- [Warp overview](https://docs.warp.dev/) · [Agents overview](https://docs.warp.dev/agents) · [Warp Agent CLI overview](https://docs.warp.dev/agents/cli) · [CLI quickstart](https://docs.warp.dev/agents/cli/quickstart) · [CLI reference](https://docs.warp.dev/agents/cli/reference) · [Oz CLI reference](https://docs.warp.dev/agents/cli/oz-cli) · [Installation and setup](https://docs.warp.dev/getting-started/quickstart/installation-and-setup) · [Changelog](https://docs.warp.dev/changelog/2026)
