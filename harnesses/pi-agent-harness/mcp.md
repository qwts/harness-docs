# MCP servers

**Verified:** 2026-11-16 (Pi v1.5.2)

## Configuration locations

“Pi connects to Model Context Protocol servers over stdio or streamable HTTP and makes their tools and resources available to the model.” “Pi reads user-level servers from `~/.pi/agent/mcp.json` and project servers from `.pi/mcp.json`. Project configuration is read only after project trust is granted. A project entry replaces a user-level entry with the same name.” A project entry without `command`, `url`, or `type` “overrides only `enabled`, `exposure`, and `toolExposure` of the user-level server with the same name and keeps the rest, including `env`, `headers`, and `auth`” — the documented way to disable a personal server in one project.

The format “matches other MCP clients,” and the docs give a conversion table for Claude Desktop/Claude Code/Cursor (“Copy the existing `mcpServers` entry”), VS Code, Codex (TOML → JSON), and OpenCode.

```json
{
  "mcpServers": {
    "filesystem": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", "."] },
    "docs": { "url": "https://example.com/mcp", "headers": { "Authorization": "Bearer ${DOCS_TOKEN}" } }
  }
}
```

## Transport and entry rules

“Stdio servers use `command`, `args`, `env`, and `cwd`. Relative `cwd` values resolve against the session directory. A leading `~/` in `command`, an argument, or `cwd` names the home directory. HTTP servers use `url`, `headers`, and `oauth` … The legacy SSE transport is not supported.” Configuration rules: server names “may contain only letters, digits, `_`, and `-`” — names differing only in `-` vs `_` “count as the same server: a second one is rejected”; `type` is optional but must be `stdio`, `http`, or `streamable-http`; “`sse` is rejected”; “`command` is one executable and `args` contains its arguments. It is not a shell command string”; `env`/`headers` values support `${GITHUB_TOKEN}` interpolation and whole-value `!command` (e.g. `"Authorization": "!echo Bearer $(gh auth token)"`); “Invalid entries are reported and skipped without preventing other servers from connecting.”

Both types accept `timeout` (“per-request timeout in seconds (default 60). Progress notifications reset it”), `enabled: false`, `exposure`/`toolExposure`, and `description` (“It lists the server in the system prompt … Without it, the first line of the server instructions is used once the server connects”).

## Shell commands and session UI

```bash
pi mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem .
pi mcp add docs --url https://example.com/mcp --bearer-token-env-var DOCS_TOKEN
pi mcp list
```

`add`/`remove` write `~/.pi/agent/mcp.json` (or `.pi/mcp.json` with `--local`/`-l`); `add` also takes `--exposure <mode>`, `--description`, `--env KEY=VALUE`, `--cwd`, `--header`, `--bearer-token-env-var`, and OAuth options. `list` is documented to “Connect to every enabled server and print its state, tools, and errors”, exiting with `1` “when a config entry is invalid or an enabled server is not connected.” `login` signs in to OAuth servers (“`--timeout` limits the whole sign-in … default 300 seconds”); `logout` will “Delete the stored OAuth credentials of a server.” “These commands work outside a session, so agents can run them through `bash`” — and “Shell commands do not load extensions.” “Project `.pi/mcp.json` files are only read for projects that are already trusted.”

In a session, `/mcp` “lists configured servers with their state, tool count, exposure, and configuration source,” supports reconnect/sign-in/exposure/enable-disable, and saves changes “to the file that defines the server without replacing unrelated content.” `/reload` applies out-of-band changes.

## OAuth

OAuth applies to “HTTP servers without an `Authorization` header”: “When the server rejects an unauthenticated connection, `/mcp` shows that it needs sign-in.” “Pi identifies itself to the authorization server with its Client ID Metadata Document or by registering a client, stores tokens in `~/.pi/agent/mcp-auth.json`, and refreshes access tokens when they expire or the server rejects them.” Details: the CIMD client ID is `https://pi.dev/oauth/client.json` with redirect `http://127.0.0.1:<port>/callback`; `clientRegistration` selects `auto`/`dcr`/`cimd`; `clientName` (default registration name `pi`), `clientId`/`clientSecret` (env or command values), `callbackPort`, `callbackUrl` (“it must use HTTP on `localhost`, `127.0.0.1`, or `[::1]`”), `scope`, and `authServerMetadataUrl` (“The URL must use HTTPS, except on `localhost`, `127.0.0.1`, or `[::1]`”) are all supported. “Credentials belong to a server name and URL.”

## Tool exposure

“Each server tool is registered as `mcp__<server>__<tool>`”; the configuration rules add that “Tools are named `mcp__<server>__<tool>`, with every character other than letters, digits, and `_` replaced by `_`; tools of a server whose names then collide all get a hash suffix.” The server's `exposure` decides how the model reaches it:

| Exposure | Behavior |
|---|---|
| `codemode` (default) | “Callable from codemode scripts, but neither declared to the model nor listed in the codemode description. Scripts find tools with `searchTools()`, `describeTool()`, or `ALL_TOOLS`.” |
| `deferred` | “Not declared until `tool_search` loads a match for the next model call.” |
| `direct` | “Declared to the model like a built-in tool and also callable from codemode.” |
| `hidden` | “Registered but unreachable.” |

`codemode-deferred` is “accepted as an alias for `codemode`.” “Pi activates `codemode` when a server with `codemode` exposure connects. It activates `tool_search` for a server with `deferred` exposure.” Per-tool overrides: “`toolExposure` overrides the server exposure for individual tools. Keys are exact server tool names or patterns where `*` matches any characters. Exact names win over patterns; among patterns, the first match wins.” Servers with indirect tools are listed in the `mcp_servers` system-prompt section. `"autoEnableCodemode": false` beside `mcpServers` prevents automatic activation, and “Pi warns once when neither `codemode` nor `tool_search` is active and non-direct tools cannot be called.”

## Runtime behavior

“Pi connects every enabled server in the background when a session starts. … The first prompt waits up to 10 seconds only for servers with `direct` tools.” “HTTP network errors and transient statuses (408, 429, and 5xx) are retried twice. A dropped connection is shown as disconnected and reconnects on the next call.” Stopping a stdio server “closes its stdin, sends SIGTERM, then sends SIGKILL to its process group.” Logging goes to `~/.pi/agent/mcp.log` (“The file moves to `mcp.log.1` after it grows past 5 MB”).

Output and resources: “Text results over 20 KB reach the model with their middle removed around a `…N chars truncated…` marker. The full text is saved to a temporary file named in the result.” Resource tools follow the Codex/OpenCode pattern — `list_mcp_resources`, `list_mcp_resource_templates`, `read_mcp_resource` — counted as MCP tools; “Resources for MCP Apps, identified by `ui://` URIs … are omitted because Pi does not render them.”

## Permissions, extensions, and SDK

“Every MCP call passes through Pi's tool pipeline. Extension `tool_call` and `tool_result` handlers, including permission gates, therefore apply to MCP tools.” Extensions can add session servers with `pi.registerMcpServer(name, config)` (“A file-configured server with the same name takes precedence”), and an installed extension that registers `/mcp` — “such as `pi-mcp-adapter`” — replaces the built-in MCP support entirely: “Pi then does not read `mcp.json` or connect its servers in a session.” Built-in MCP can also be disabled via `-builtin:mcp` in settings or `--no-mcp` for one run. SDK sessions “do not load built-in extensions. Add the MCP extension, the codemode extension for `codemode` servers, and the tool-search extension for `deferred` servers to the resource loader.”

## Official sources

- [MCP Servers](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/mcp.md) · [Command Line (MCP commands)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md) · [Codemode](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/codemode.md) · [Security (project trust)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md)
