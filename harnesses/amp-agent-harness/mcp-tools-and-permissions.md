# MCP, tools, and permissions

**Verified:** 2026-11-15

## The approval default is inverted

“By default, Amp does not ask for approval before running tools.” The documented control points are plugins (intercepting `tool.call` with allow/reject/modify/synthesize), the `amp.tools.disable` setting, MCP permission rules and per-tool policies, and enterprise enforcement. The tools page’s own safety guidance is to treat untrusted inputs seriously: “Untrusted repositories, MCP servers, and other external inputs can influence what Amp does,” with a custom policy plugin or an isolated development environment as the suggested mitigations.

`amp tools list` shows the built-in tools; the docs do not enumerate them on that page. Painter, the image generation/editing tool, is documented separately (GPT Image 2.5 Sunburst, one image per call, up to three reference images).

## MCP configuration

Servers are configured under `amp.mcpServers` in user (`~/.config/amp/settings.json`) or workspace (`.amp/settings.json`) settings, with stdio entries (`command`, `args`, `env`) and remote entries (`url`, `headers`). `${VAR_NAME}` interpolation works in config values, and `${amp:id-token}` may appear in remote-server headers in orbs (HTTPS servers without OAuth only; rejected in URLs and local-server fields). The CLI manages servers with `amp mcp add <name> -- <command>` or `amp mcp add <name> <url>`; remote definitions are stored by ampcode.com and scoped with `--personal`, `--workspace`, `--project`, or `--current-project`.

Loading precedence, highest first: CLI `--mcp-config`, then workspace settings, then user settings, then skills — “you can override skill-provided MCP servers with your own configuration.” Servers with the same ID override each other (project over workspace, workspace over personal); disabled servers do not override.

## MCP trust and policies

Two independent layers apply:

- **Server-level trust:** workspace-configured servers (`.amp/settings.json`) “require explicit approval before running — this prevents untrusted code from executing automatically when you open a project.” Pending servers show in `amp mcp doctor` and are approved with `amp mcp approve <name>`. User-level config and `--mcp-config` entries need no approval.
- **Server permissions:** the `amp.mcpPermissions` setting holds allow/reject rules matched against local servers’ `command`/`args` and remote servers’ `url`; “Amp applies the first matching rule,” and unmatched servers are allowed.
- **Per-tool policies (remote servers):** `Allow` (default), `Ask First`, or `Disabled`, plus a forced Ask First for tools advertising `anthropic/requiresUserInteraction: true`.

## Enterprise enforcement

- **Allowed models:** workspace admins disable model groups (open-weights, Chinese labs, SpaceXAI/Grok, Meta Muse, Claude Fable); members cannot pick disabled models in the selector or CLI, running threads stop at the next message with “Model Not Available,” and group grants can only add access, never re-enable an off group. The workspace’s new-thread default and the models Puck and the Spaces agent run on are locked and cannot be disabled.
- **MCP registry allowlist:** enterprise admins can enforce a registry so “only approved MCP servers are available to workspace members.” The machine that starts a server checks it against `registry.modelcontextprotocol.io`; remote servers are checked by ampcode.com, local ones by the CLI or runner, and orb threads by the orb. “If a machine cannot reach the registry, it blocks every MCP server it would start.” Local matching is by package name across npm, Python (uvx/pipx), and Bun runners, with documented blocked workarounds (`--from`, `--with`, alternate indexes, aliases, direct URLs, env-var overrides like `NODE_OPTIONS` or `npm_config_*`).
- **Minimal data retention:** on Amp-managed connections, prompts and responses are retained by providers only as long as needed to respond, with documented exceptions (safety monitoring, “up to 30 days when specified” for some models). “Neither Amp nor its model providers use customer data to train models, unless you have explicitly opted into training,” and training “can never be enabled on Enterprise workspaces.” MDR “does not control how Amp stores thread data.”

## Thread visibility controls

Without a workspace, threads are private; in workspaces, threads in workspace-owned projects are shared with the workspace by default. Visibility levels are Private (“You, plus workspace admins through audited access”), Workspace, Group (Enterprise), and Unlisted (“Anyone on the internet who has the link”). Admin access to private threads is re-authenticated, shows an audit notice, and is audit-logged, with read-only artifacts.

## Official sources

- [Tools](https://ampcode.com/docs/tools)
- [MCP](https://ampcode.com/docs/customize/mcp)
- [Enterprise: allowed models](https://ampcode.com/docs/enterprise/allowed-models)
- [Enterprise: MCP registry allowlist](https://ampcode.com/docs/enterprise/mcp-registry-allowlist)
- [Enterprise: minimal data retention](https://ampcode.com/docs/enterprise/minimal-data-retention)
- [Threads](https://ampcode.com/docs/threads)
