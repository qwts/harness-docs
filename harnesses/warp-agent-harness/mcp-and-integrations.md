# MCP servers and integrations

**Verified:** 2026-11-16 (Warp docs `warpdotdev/docs` main)

## MCP in the Warp app

“MCP servers extend Warp's local agents in a modular, flexible way by exposing custom tools or data sources through a standardized interface … Warp supports a variety of connection protocols, including Streamable HTTP and SSE, along with custom headers and environment variables.”

- **Adding servers** — the **+ Add** button accepts a startup command (`command`, `args`, optional `env` and `working_directory` — “Always set `working_directory` explicitly when your MCP server command or args include relative paths”), a URL (`url` + `headers` for Streamable HTTP/SSE), or a pasted JSON `mcpServers` snippet.
- **Lifecycle** — “If you close Warp with an MCP server running, it will run again on next start of Warp. MCP servers that are stopped will remain so on next launch.” `/mcp`-style management lives in **Settings** > **Agents** > **MCP servers**; **View Logs** helps debugging (“make sure to remove any sensitive information before sharing, as they may contain API keys”).
- **Auth** — “Environment variables”, “OAuth login (one-click installation)” (“Warp stores credentials securely on your device and reuses them for future sessions. Re-authentication is required when opening Warp on a new machine”), or “Custom Headers”.
- **Sharing** — teammates install shared servers; “sensitive values in the `env` configuration will be automatically scrubbed and replaced with variables.”

## File-based MCP config (app)

“File-based servers are defined in config files that Warp detects and can spawn automatically,” managed via the bundled `/agent-add-mcp` skill. Warp reads its own files plus third-party providers' configs:

| Provider | Global config | Project-scoped config | Auto-spawn |
|---|---|---|---|
| Warp | `~/.warp/.mcp.json` | `.warp/.mcp.json` at project root | On by default |
| Claude Code | `~/.claude.json` | `.mcp.json` at project root | Requires toggle |
| Codex | `~/.codex/config.toml` | `.codex/config.toml` at project root | Requires toggle |
| Other agents | `~/.agents/.mcp.json` | `.agents/.mcp.json` at project root | Requires toggle |

“Global Warp servers auto-spawn by default. Global servers from all other providers auto-spawn only when **Auto-spawn servers from third-party agents** is enabled. Project-scoped servers from any provider require explicit approval.” Project-scoped servers are “session-scoped — after restarting Warp, toggle them on again if you still trust the repo.”

Security gates: “**Config edits require approval** - Warp prevents edits to MCP config files unless you explicitly approve the change. This keeps an agent or automation from silently adding a server that can run commands or access data,” and “**Project-scoped servers never auto-spawn** - Warp detects project-scoped MCP config files in cloned repos, but requires you to start each server manually. This prevents a cloned repo from automatically starting an MCP server that runs arbitrary local commands.”

## MCP in the Warp Agent CLI

“The CLI keeps its own MCP server configuration, separate from the Warp app's, so each can run its own set of servers.” Config is JSON “using the same `mcpServers` format” (macOS file: `~/.warp_cli/.mcp.json`); “Configured servers start automatically once you're logged in,” and edits are picked up automatically.

Scope restriction: “The CLI reads MCP servers from its global config file only. Project-scoped MCP config files in repositories are not detected. To copy global server definitions from the Warp app, use the `/tui-migrate-setup` bundled skill.” `/mcp` opens the management view: “Each configured server is listed with its transport (`stdio` or `HTTP/SSE`) and current status, including the number of tools a running server exposes”; `Enter` starts/stops/retries, and “servers awaiting authentication reopen the OAuth page in your browser.”

## Platform integrations (cloud side)

Cloud agents consume triggers and context through platform integrations — GitHub, Bitbucket, GitLab, Azure DevOps, Jira, Linear, Slack, GitHub Actions, and cloud providers — configured in the web app; “The CLI integration setup guide covers their configuration” for the legacy Oz CLI. “Integrations supply work and context from tools your team already uses. For example, a message or issue can become a task for an agent.” Secrets are platform-managed: “Secrets give agents credentials for resources such as package registries, APIs, or databases. Grant only the access needed for the task,” with factory secrets and harness authentication as separate references.

## ACP/API/SDK

There is no ACP surface in the Warp docs (unlike some peer harnesses); the programmatic paths are the **Warp Platform API** and SDKs — “an internal service submit work and inspect results programmatically,” with “agent and run endpoints for standalone runs and run inspection” — plus the Oz CLI for scripts/CI and `WARP_API_KEY` for headless CLI login. The Conversation API exposes `read_skill` actions with `bundled_skill_id` / `skill_path` inputs for measuring skill usage.

## Official sources

- [MCP (app)](https://docs.warp.dev/agents/capabilities/mcp) · [Platform overview](https://docs.warp.dev/platform/overview) · [Integrations](https://docs.warp.dev/platform/integrations) · [Secrets](https://docs.warp.dev/platform/secrets) · [API and SDKs](https://docs.warp.dev/factories/api-and-sdk) · [Agent Memory](https://docs.warp.dev/agents/agent-memory)
