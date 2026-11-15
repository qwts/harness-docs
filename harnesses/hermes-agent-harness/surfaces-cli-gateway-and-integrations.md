# Surfaces: CLI, gateway, Bot Mode, and integrations

**Verified:** 2026-11-15

## Install and setup

Primary install: `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`; the overview also documents a PowerShell path (`irm …/install.ps1 | iex`), Termux (APT), Nix, and Docker. `hermes update` keeps an install current. First-run setup is one command — `hermes setup --portal` — then `hermes chat`. Setup can seed bundled skills or pick the Blank Slate path (`hermes skills opt-out` on non-interactive installs; `hermes profile create <name> --no-skills` for named profiles).

## Classic CLI — a gateway client

`hermes chat` (and bare `hermes`) “no longer runs the agent inside your terminal process”: it discovers or starts the profile’s gateway and attaches over a local WebSocket — the same session a Desktop window, the TUI, or a Telegram topic can open at once. Closing the terminal detaches (“Detached; accepted work continues at the gateway.”); `/exit`, `/quit`, `/new`, and Ctrl+D close the chat and stop background subagents. Launch flags (`--model`, `--toolsets`, `--yolo`, …) “are frozen into the session at creation; `--resume` may repeat the same values but cannot change them.” A finite query whose owner is lost mid-queue exits with code 3, input still queued — “Do not resend the queued input.”

Notable interactions: `Esc Esc` backtrack (transcript browser, edit-a-prior-prompt rewinds the session, `f` forks); `Ctrl+T`/`F6` live work dock; `Ctrl+S` prompt stash; `Ctrl+G` editor compose; `!` shell mode (“The model is never invoked” — runs on the host, not the terminal backend); `/busy interrupt|queue|steer` controls mid-run input; `/queue` and `/steer` manage follow-ups explicitly; `/worktree new` creates in-session worktrees; `/handoff <platform>` moves the session to messaging with full transcript replay.

## TUI and Desktop

`hermes --tui` opens the modern TUI — “modal overlays, mouse selection, and non-blocking input” — with a live session switcher (`/sessions new` starts another). The Desktop app hosts the Bots pane, skills browser, MCP page, and multi-machine **Connections**.

## Bot Mode

“Bot Mode ships built into the desktop app and is on by default”, and “a Bot **is** a Hermes profile” — isolated config, memory, skills, and credentials under `~/.hermes/profiles/<name>/`. Each Bot gets a canonical, persistent **Bot Chat**: `/new` inside it “would fork the relationship into a scratch session — the one thing Bot Mode promises never happens”, so the composer reroutes it to `/compact`. Routines are plain cron jobs namespaced `[bot:<name>]`; `hermes -p <bot> chat` opens the same agent from a shell.

Bots message each other with the `message_agent` tool (canonical Bot Chats only; the install must be Bot-Mode-managed), join **group chats** (2–6 Bots; up to three serial rounds per message, 10-message cap, `@name`/`@user`/`@everyone` mentions, a *needs you* badge for escalations), and can be seated across machines — the Desktop relays between registered Connections, and `hermes peer add <name> --url … --key <API_SERVER_KEY>` registers a gateway-to-gateway peer for Desktop-less DMs (`hermes peer dm|run|status|stop`). A loop guard caps bot-to-bot exchanges (`gateway.bot_loop_guard.max_hops`, default 4); replies are notify-only so “two Bots cannot keep each other busy forever.”

## Messaging gateway

`hermes gateway run` (or `hermes gateway install` as a user service; `--system` for boot-time) serves 20+ platforms — Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Mattermost, Email, SMS (Twilio), Home Assistant, DingTalk, Feishu/Lark, WeCom, Weixin, BlueBubbles (iMessage), QQ Bot, Teams, and more. `/sethome` marks a chat as the delivery target; `/pause` is the in-chat global stop; `/platform list|pause|resume` operates adapters; `/restart` drains and restarts. Home-channel targets feed cron delivery (`TELEGRAM_HOME_CHANNEL`, `DISCORD_HOME_CHANNEL`, …).

## Headless and one-shot

`hermes chat -q "prompt"` for single queries; `--query-file` (or stdin) passes arbitrary text verbatim — “nothing is shell-interpreted”; one-shot runs accept `-s` skill preloads and `--toolsets`. Runtimes that cannot receive detached results (`hermes -z`, CLI cron runs) “fall back to synchronous execution automatically.”

## ACP

`hermes acp` (equivalently `hermes-acp` or `python -m acp_adapter`) speaks Agent Client Protocol over stdio — “logs to stderr, keeping stdout reserved for ACP JSON-RPC traffic.” Requires the `acp` extra (`pm.sync_venv(['acp'])`). Editors get chat, tool activity, file diffs, approval prompts, and streamed chunks; the curated `hermes-acp` toolset “intentionally excludes things that do not fit typical editor UX, such as messaging delivery and cronjob management.” Editor-supplied MCP servers “are always added”; tool schemas freeze for the conversation. Clients: the VS Code ACP Client extension, Zed custom agent server (`hermes`, `["acp"]`), JetBrains ACP plugins, and the Buzz relay (which answers permission prompts itself with `allow_once` — a documented caution). Approvals map to allow once/session/always/deny, failing closed on timeout.

## API server and peers

The `api_server` gateway platform exposes REST/streaming endpoints (`/v1/chat/completions`, `/v1/responses`) guarded by `API_SERVER_KEY`, powers `hermes peer` cross-machine calls, and can host group-chat members for other gateways (approvals must not be `off`).

## Product separation

- **Hermes Agent ≠ Hermes models.** The harness is an agent application (MIT, `github.com/NousResearch/hermes-agent`) with its own docs site; the Hermes model family is a separate Nous product the harness documentation does not cover. The agent routes to many vendors’ models through its provider registry.
- **Messaging bots vs Bot Mode vs subagents.** The docs distinguish messaging-platform bots (gateway adapters), Bot Mode Bots (profiles with canonical chats), and delegated subagents (`delegate_task` children) — “Profiles, agents, and bots” are related but distinct layers.
- **Org identity.** The `hermes` registry identity (with its aliases) normalizes to this single harness.

## Official sources

- [Overview](https://hermes-agent.nousresearch.com/docs) · [CLI Interface](https://hermes-agent.nousresearch.com/docs/user-guide/cli) · [TUI](https://hermes-agent.nousresearch.com/docs/user-guide/tui) · [Hermes Desktop](https://hermes-agent.nousresearch.com/docs/user-guide/desktop) · [Bot Mode](https://hermes-agent.nousresearch.com/docs/user-guide/bot-mode) · [ACP](https://hermes-agent.nousresearch.com/docs/user-guide/features/acp) · [Slash Commands](https://hermes-agent.nousresearch.com/docs/reference/slash-commands) · [llms.txt](https://hermes-agent.nousresearch.com/docs/llms.txt)
