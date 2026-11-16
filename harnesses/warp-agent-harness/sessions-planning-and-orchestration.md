# Sessions, planning, orchestration, handoff, and cloud automation

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main)

## App: Terminal mode and Agent Mode

“Warp provides two distinct modes: a clean terminal for commands, and a dedicated conversation view for multi-turn agent workflows.” “Terminal mode keeps the interface minimal by default, while Agent Mode surfaces full controls for model selection, voice input, image attachments, and conversation management.” The hint bar bridges them (`⌘↩ to send to agent`, error-block attachment, `⌘Y to continue conversation`), with optional auto-detection that labels input as “agent” or “shell” before submit.

## CLI sessions and conversations

“Type a prompt in plain language, such as `What does this project do?`, and press `Enter`.” The transcript renders “streamed responses, Markdown formatting, file-edit diffs, tool calls, plans, and task lists”; tool calls render as “a one-line status row with a state glyph,” diffs nest under “`Edited 3 files`” summaries, thinking streams into a collapsible `Thinking...` section, and multi-step work tracks a `≡ Tasks` list with `◌`/`●`/`✓` glyphs.

- **Shell mode** — “Type `!` at the start of an empty input to switch to shell mode”; `/natural-language-detection` classifies input as you type so “recognized commands switch to shell mode, while natural-language prompts stay in Agent Mode.”
- **Exit/resume** — “Press `Ctrl+C` twice in a row to exit, or run `/exit`. A single `Ctrl+C` press stops the agent's current response.” On exit “the CLI prints a resume command for the conversation”: `warp --resume CONVERSATION_TOKEN`.
- **Persistence** — “Conversations save to your Warp account, so you can exit and resume later or switch between them.” `/conversations` (or `←` on empty input) browses history; `/fork` copies a conversation; `/compact [instructions]` summarizes to “Free up context”; `/export-to-clipboard` and `/export-to-file` dump Markdown (over SSH, clipboard copy “using OSC 52 escape sequences”).
- **Planning** — “To have the agent first research and produce a plan before making changes, use the `/plan` slash command”; plans render inline with a status header, `Ctrl+Shift+P` collapses/expands. The app's native planning adds “a persistent plan editor, version history, selective execution” — “Any update made by the agent **creates a new version**.”

## Multi-agent orchestration

“Orchestration lets one conversation spawn child agents that work in parallel, on your machine or in the cloud.” Trigger it by asking (“split this migration across three agents”) or `/orchestrate [task]`.

- **Approval gate** — “When the agent decides to spawn child agents, the CLI shows an approval card titled **Can I start additional agents for this task?** … By default, no child agents launch until you approve. The **Run orchestrated agents** permission in your profile controls this: **Always allow** skips the approval card, and **Never** blocks child agents entirely.” `Ctrl+E` edits configuration before launch (local vs cloud, model, and for cloud children the harness, environment, and host).
- **Locality** — “Local child agents - Each local child runs on your machine as its own session, starting in the parent's working directory. Local children always run the Warp Agent.” “Cloud child agents - Each cloud child runs in a cloud environment, and can run a different harness than the parent, such as Claude Code or Codex.” Cloud children “only show status” in the CLI.
- **UI** — “a tab bar labeled **Agents:** appears above the session with one tab for the orchestrator and one tab per child agent”; `Ctrl+C` kills the selected child. On the platform side, “A parent running the Warp Agent analyzes a task and breaks it into subtasks,” can dispatch “Claude Code subagents for steps that need careful review and Codex subagents for high-volume edits,” and factory configs can dispatch named agents.

## Handoff (local → cloud)

“Handoff promotes your Warp Agent CLI conversation into a cloud agent run.” What moves: “**Conversation history** - Warp forks the conversation, so the cloud agent sees the full transcript and your local conversation stays untouched”; “**Workspace changes** - Warp snapshots your uncommitted repository changes, including untracked files, and applies them in the cloud run”; “**Pending input** - The optional follow-up prompt and any images attached to the input move to the cloud run.” Run `/handoff [prompt]`; “The card suggests your most recent environment (preferring one that contains the current repository) and preselects your conversation's current model if it can run in the cloud.” Constraints: “You can't hand off while a command is still running, or while the conversation has child agents that are in progress or blocked,” and “Handing off an orchestrator forks only its own conversation. The orchestration and its child agents stay local.”

Resuming cloud work: in the browser (Oz web app) or in the CLI — “Pull a finished cloud run into your terminal … Your next prompt continues the conversation locally.” Two gates: “The run must be finished” and “Warp Agent runs only - Conversations that ran with a different harness, such as Claude Code or Codex, can't be opened in the CLI.”

## Cloud agents, triggers, and schedules

Cloud agents run on the Automation Platform: “A trigger starts work: a schedule, a message in a connected tool, or a request from the app, CLI, or API.” “An environment groups a container image, repositories, and setup commands, such as installing the dependencies an agent needs to run tests,” and hosts are either “Warp-hosted execution uses compute provisioned by Warp” or “Managed self-hosting uses a worker on your infrastructure.” **Scheduled Agents** “run cloud agents automatically on a **recurring schedule**” — “Runs automatically based on a cron expression,” “Uses a fixed prompt defined at schedule creation time,” and “Starts a fresh agent session for every run.” Slack/Linear triggers, GitHub Actions, and the Platform API cover other entry points.

Computer Use belongs to this world: “Computer Use enables agents to interact with desktop environments … all within a secure, isolated sandbox,” “only available in Warp's sandboxed cloud environments, not in local interactive terminal sessions,” and “**enabled by default** for cloud agent runs on Warp's built-in harness” (disabled by default on third-party harnesses). It can “Drive the bundled Chromium browser, visually or with scripted Playwright commands” and attach screenshots/recordings to PRs.

## Warp Factories (boundary)

“Warp Factories is open, flexible infrastructure for building, running, measuring, and improving your own software factory. Define workflows as code, connect your repositories and existing tools, and use evaluation and benchmarks to compare configuration changes.” Factories and standalone cloud agents “run on the Automation Platform … It provides the execution, configuration, and APIs they share.” Inside a factory definition, the built-in agent is selected with `harness.type: oz`; “The Warp Agent is Warp's built-in harness for factory agents and standalone cloud runs. It runs commands, edits code, and coordinates work across models and tools,” needing “no third-party auth secret.” Team admins “can disable any harness for their workspace.”

## Third-party CLI agents (adjacent feature)

Separately from the Warp Agent, “Warp gives third-party CLI coding agents first-class support. Run Claude Code, Codex, OpenCode, and others inside Warp and get rich input, agent notifications, inline code review, and remote session control” — sixteen agents including Amp, Auggie, Copilot CLI, Cursor CLI, Gemini CLI, Droid, Pi, Goose, Antigravity, Hermes, Mistral Vibe, Grok Build, and Kiro CLI. Their sessions “run inside a terminal pane … not as Warp Agent conversation threads”: “Claude Code, Codex, OpenCode, and other third-party CLI agents do **not** appear there” in the Conversation Panel. **Remote Control** publishes such a session: “monitor progress, review output, and steer the agent from your phone, a web browser, or another computer.” Claude Code and Codex are “also supported as harnesses in the Automation Platform for multi-agent orchestration.”

## Official sources

- [Terminal and Agent modes](https://docs.warp.dev/agents/local-agents/interacting-with-agents/terminal-and-agent-modes) · [Agent conversations (CLI)](https://docs.warp.dev/agents/cli/agent-conversations) · [Cloud handoff and orchestration](https://docs.warp.dev/agents/cli/cloud-and-orchestration) · [Planning](https://docs.warp.dev/agents/capabilities/planning) · [Handoff](https://docs.warp.dev/platform/handoff) · [Scheduled Agents](https://docs.warp.dev/platform/triggers/scheduled-agents) · [Automation Platform overview](https://docs.warp.dev/platform/overview) · [Warp Agent harness (Factories)](https://docs.warp.dev/factories/harnesses/warp-agent) · [Third-party CLI agents](https://docs.warp.dev/agents/cli-agents/overview) · [Computer Use](https://docs.warp.dev/agents/capabilities/computer-use)
