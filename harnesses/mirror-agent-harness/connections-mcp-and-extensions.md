# Connections, MCP, and extensions

**Verified:** 2026-11-15

## Connections (built-in services)

“Connections give the agent tools for services outside your project. … Each connection is an MCP (Model Context Protocol) server, and once it's connected its tools work like the agent's built-in ones.” “Mirror knows how to connect Notion, Datadog, Linear, and Slack.” Flow: “Run `/connect` and choose a service, or name it directly, as in `/connect linear`. … Sign in to the service in your browser. Mirror opens the sign-in page and also prints its link.” “Mirror saves the sign-in under `~/.mirror/mcp/credentials/`, so you only sign in again if it expires or the service rejects it.”

Management: `/connections` (“Show connected services and their status”), `/connect [service]`, and `/disconnect [service]` — “This deletes its definition file in `~/.mirror/mcp/` and its saved sign-in. Without a name, choose from a list.” Tool naming: “The agent sees each tool as `mcp__<service>__<tool>`, such as `mcp__linear__create_issue`. Tool calls from connections need your approval when the workspace uses manual approval.”

## Custom MCP servers

“To connect an MCP server that Mirror doesn't list, describe it in a JSON file in `~/.mirror/mcp/`, then run `/connect <name>`. The file name is the server's name: `~/.mirror/mcp/my-server.json` is connected with `/connect my-server`. Names can contain letters, digits, `-`, and `_`. Each file describes one server and sets either `url` or `command`.” The security posture is project-hostile by design: “Mirror only reads server definitions from this directory, never from a project, so cloning a repository can't add servers. To keep definitions somewhere else, set `MIRROR_MCP_DIR`.”

Remote servers set `url` with `auth` of type `oauth` (optionally `clientId`) or `bearer` with `tokenEnvVar` naming “the environment variable that holds it … The key itself never goes in the file.” “A server that uses authentication must use HTTPS. The exception is a server on your own machine (`localhost`, `127.0.0.1`, or `::1`).” Local servers set `command` and `args`, and `env` values are indirections: “each value is the name of one of your environment variables, not the secret itself.”

Optional fields: `enabledTools` (“Expose only the tools listed … Omit it for every tool, or use `[]` for none”), `startupTimeoutSeconds` (“Defaults to 15”), `toolTimeoutSeconds` (“Defaults to 300”). “If Mirror can't start a server, `/connect` reports it and names the file to check. Changes to a server that's already connected take effect the next time you start Mirror.” Note the destructive disconnect: “`/disconnect` deletes the server's JSON file. Keep a copy of any custom definition you may want to reconnect later.”

## Web search (not MCP)

“The agent's web search and page-reading tools use a web provider directly, with its API key in your environment. They don't need an MCP connection.” “Mirror uses You.com for web search and page reading by default. You can choose Tavily or Firecrawl instead with `/web-provider`. This choice is separate from your model provider.”

| Web provider | Environment variable | Selection |
|---|---|---|
| You.com (default) | `YOU_API_KEY` | `/web-provider you` |
| Tavily | `TAVILY_API_KEY` | `/web-provider tavily` |
| Firecrawl | `FIRECRAWL_API_KEY` | `/web-provider firecrawl` |

“Exporting a key alone doesn't select that provider.” The choice is saved as `web_provider` in `~/.mirror/config.json`; “Calls already running finish with their original provider. Without a saved choice, Mirror uses You.com.” Limits: “You can use Mirror for coding tasks without a web provider key. Web search and page reading are available when the selected provider's key is set. Mirror doesn't fall back to another provider if a call fails.” Web provider keys are separate from `REFLECTION_API_KEY`.

## Extensions

“Extensions add slash commands, tools for the agent, and widgets on the Mirror screen. Much of Mirror is built this way: the model picker, connections, the repository line, and memory are all bundled extensions.” The extension API is “in early beta and may change between releases.”

### Bundled extensions

Eighteen ship with Mirror: `inference_controls` (`/model`, `/effort`, `/login`, `/logout`), `settings` (`/settings`, `/approvals`), `connections`, `memory`, `context_usage` (`/context`), `session_usage` (`/usage`), `web_provider`, `feedback`, `tokens_per_second`, `repository_status`, `editor` (`/editor`, `Ctrl+P`), `external_editor` (`Ctrl+G`), `themes`, `tips`, and four off by default: `view_mode`, `vim` (“Vi-style prompt editing”), `todo` (“A task list that the agent keeps above the prompt while it works”), and `user_extensions` (“A tool that lets the agent write extensions for you”). Commands provided by bundled extensions “disappear if you turn their extension off.”

### Enabling, installing, and consent

`/extensions` opens the picker: “Type to filter it, use the arrow keys to select an extension, and press `Enter` to turn it on (`●`) or off (`○`).” It also shows “extensions awaiting approval (`◐`) and failed extensions (`!`), with the reason.” “Turning an extension off removes its commands, tools, and widgets from every workspace straight away. Mirror saves your choices in `~/.mirror/config.json`.” Mirror loads an extension's code once — after editing, “turn it off and on again in `/extensions`, or restart Mirror.”

Discovery locations: `~/.mirror/extensions/<name>/extension.py` (“In every project”) and `.mirror/extensions/<name>/extension.py` (“When you start Mirror in this repository. Mirror checks every directory from the repository root down to the launch directory”). “Mirror only looks for extensions at startup.” Before importing new external extensions Mirror “shows their source paths in an approval list” with **Approve selected** / **Approve all** / **Reject all**; “Press `Escape` to skip the batch for this launch. Mirror remembers approval and rejection choices.” The bottom line: “An extension is Python code that runs inside Mirror with your account's permissions. Approval mode doesn't restrict it.”

Install paths: `/extensions-add <GitHub URL>` (“the HTTPS URL of a repository that has an `extension.py` at its root”), `/extensions-add <local directory>` (paths “refer to the CLI host, and relative paths start from the directory where you launched Mirror, even if you've switched workspaces”), `/extensions-update` (“only updates GitHub sources”), `/extensions-remove` (unregisters “without deleting its files or saved settings”). “Updates are explicit; Mirror doesn't replace a running extension just because its remote repository changed.” Launch switches: `--trust-extensions` and `MIRROR_EXTENSIONS_ENABLED=false` (“Set to `false` to skip all extensions except bundled ones”).

The `user_extensions` tool manages extensions in `~/.mirror/extensions/` with actions `list`, `create` (“Write a starter extension, without loading it. Its ID is its directory name”), `load`, `reload`, and `unload`; “In manual approval mode, each use of the tool needs your approval.”

### Python API

“An extension is a Python class that subclasses `mirror.cli.extension.Extension`. Mirror creates one instance and shares it across all your workspaces, then calls methods on it to find out what it provides”: `tools(context)` returns agent-callable tools, `slash_commands()` returns commands whose handlers produce actions that `reduce()` applies to per-workspace state, `render()` draws widgets, and slow work goes in `handle_effect()`. “An extension module must define exactly one concrete `Extension` subclass, with a constructor that takes no arguments,” and `id()` must match the directory name — “Mirror rejects an ID that doesn't match.” Other overridables: `description()`, `keybindings()` (“`Escape` on its own is reserved”), `initial_state()`, `render_region()` (`STATUS_LEFT`, `STATUS_RIGHT`, `WIDGET_ABOVE_PROMPT`, `WIDGET_BELOW_PROMPT`), `on_load`/`on_unload`, `on_config`, `on_foreground`/`on_background`, `default_enabled()`, `default_config()`.

Tool details: agent-side names are built from the extension ID (`glossary__lookup`), `ToolDefinition` takes `needs_approval` (“Leave the default, `True`, for any tool that changes something”) and `timeout_seconds` (“which defaults to 300”). State: “Mirror shares one instance of each extension across all workspaces, but keeps a separate state for each workspace. … State resets when a workspace opens a conversation. To keep it, set `reset_on_session_open = False`.” Commands can set `available_when_busy=True` to run “while the agent is working; commands without it wait for the turn to finish.”

Config keys: “Bundled extensions use `builtin/<name>` keys. Discovered extensions use the absolute path of their `extension.py` file. Sources registered with `/extensions-add` use their GitHub URL or local directory path.” Each entry is `true`/`false` or an object with `enabled` and a `config` object that “overrides the extension's `default_config()`.” Failure behavior: “If an extension fails to load, Mirror leaves it out and logs the error. If it raises an error while running, Mirror disables it in that workspace and shows the error above the prompt.” Extension errors land in “the `cli-stderr` log under `~/.mirror/logs/<conversation-id>/`.”

## Official sources

- [Connections and MCP](https://developers.reflection.ai/mirror-mcp.md) · [Extensions](https://developers.reflection.ai/mirror-extensions.md) · [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md) · [Mirror CLI overview](https://developers.reflection.ai/mirror-overview.md)
