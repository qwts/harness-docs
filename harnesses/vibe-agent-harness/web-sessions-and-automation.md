# Vibe Code Web and automation surfaces

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site main)

## Vibe Code Web

“Vibe Code Web is the control plane for remote coding agents.” Entry points: “From the web: click **New session** inside a project” or “From the Vibe CLI: prefix a prompt with `&` to send it to a cloud session.”

- **Projects** — “A project is the workspace Vibe Code Web organizes everything around: one or more GitHub repositories, the sessions you run against them, and the history of past runs.”
- **Sessions** — “A **session** is one run of the Vibe Code agent against a project: a sandbox provisioned for the task, the agent loop that does the work, and the branch or pull request it leaves behind.” The agent loop: “**Read** … **Plan** … **Edit** code or **run a command** … **Report** … **Ask** if it needs clarification, then **wait** for your reply,” continuing “until the task reaches a natural stopping point, the agent asks a question, or a limit is reached.”
- **Sandbox** — “Each Vibe Code Web session runs in an **isolated cloud sandbox**”: “Single-tenant for the duration of the session … Deleted when the session ends.” “The sandbox **cannot reach your local machine**.” Default image: “a default Linux image with a baseline of common developer tooling” (POSIX shell, Git with the Mistral GitHub App credentials, common runtimes); “Custom sandbox configuration is a follow-up capability.” Network: “**outbound internet access managed by Mistral** … Inbound access from the public internet to the sandbox is not exposed,” with outbound rate limits.
- **GitHub** — “The clone uses the Mistral GitHub App's user access token, so all subsequent Git operations (commits, branches, pushes) are attributed to you.” “Repository-level actions (creating branches, pushing commits, opening pull requests) follow the GitHub permissions you granted.”
- **Security** — prompt injection is treated as a first-class risk: “**Untrusted content can attempt to redirect the agent's behavior**.” Mitigations: “All commands and file changes are visible in the **activity stream**” and “GitHub **branch protections, required reviews, required status checks, and `CODEOWNERS`** still apply to whatever the agent pushes.” Data handling: “Files in the sandbox … are processed for the duration of the session … Anything that must survive the session has to be committed to the repository,” with session activity retained “according to your plan and the applicable Mistral data policy.”
- **Slack** — a documented quickstart integrates Vibe Code Web with Slack (“AI-generated content: This app uses AI to generate responses and code.”).

## Automation boundaries

- **Programmatic mode** — `vibe --prompt` runs headless: interactive tools disabled, “Vibe denies every tool call that needs approval” unless `--auto-approve`/`--yolo`, and “**Programmatic mode never prompts for trust**” (`--trust` for one run).
- **Scheduled prompts** — `/loop` schedules “a recurring prompt in plain language” within a CLI session; “Ask the agent to list or cancel schedules.”
- **Background processes** — `enable_background_processes` (default true) controls whether the agent may “run commands in the background”; otherwise “commands run to completion in the foreground.”
- **Cloud delegation** — `&` sends a prompt to a cloud session from the CLI; cloud sessions otherwise follow the same lifecycle as web-created ones.
- **No daemon/wakeup claims** — the docs expose no standalone daemon or machine-wakeup surface for Vibe Code; long-running autonomous execution lives in Vibe Code Web sessions and Slack entry points (verified absence, not invented).

## Mode boundary

“Vibe Code is built primarily for source code work (files, repositories, commands, diffs, pull requests). For tasks that live across apps, documents, chats, meetings, or business tools, Vibe Work is usually a better fit.” Vibe Work is a separate productivity mode (skills, connectors, libraries, scheduled tasks, canvas, mini apps) sharing the same account and connectors; this reference does not document it beyond the boundary.

## Official sources

- [Vibe Code overview](https://docs.mistral.ai/vibe/code/overview) · [Get started (Web)](https://docs.mistral.ai/vibe/code/vibe-code-web/get-started) · [Sessions](https://docs.mistral.ai/vibe/code/vibe-code-web/sessions) · [Sandbox environment](https://docs.mistral.ai/vibe/code/vibe-code-web/sandbox-environment) · [Security](https://docs.mistral.ai/vibe/code/vibe-code-web/security) · [Limits and lifecycle](https://docs.mistral.ai/vibe/code/vibe-code-web/limits-and-lifecycle) · [Slack integration](https://docs.mistral.ai/vibe/code/vibe-code-web/slack-integration) · [GitHub permissions](https://docs.mistral.ai/vibe/code/vibe-code-web/github-repositories-permissions)
