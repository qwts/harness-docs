# MCP, hooks, plugins, and connectors

**Verified:** 2026-10-29

## MCP configuration

Factory supports these MCP transports:

| Transport | Meaning |
|---|---|
| `stdio` | Local server process |
| `http` | Streamable HTTP |
| `sse` | Legacy standalone HTTP+SSE |

Configuration can come from:

```text
~/.factory/mcp.json
ancestor .factory/mcp.json
project .factory/mcp.json
org-managed configuration
```

`droid mcp add` and registry additions write to user configuration. A project server cannot be removed through `droid mcp remove`; toggling one writes a user-level copy rather than modifying the project file.

Project `mcp.json` must not contain secrets. `${NAME}` expansion is limited to credential-bearing fields:

- stdio `env`;
- remote `headers`;
- OAuth client ID and secret fields.

It does **not** expand stdio `command`, `args`, or a remote `url`. This prevents using general environment substitution as a hidden command or endpoint templating mechanism, but configured environment and header values can still contain credentials and must be protected.

### Tool filtering and organization policy

`enabledTools` and `disabledTools` filter model-visible tools. If a tool appears in both, it remains excluded. Filtering visibility is not a sandbox for the server process: a stdio server is still local executable code with whatever access its process receives.

`mcpPolicy` matches a remote hostname or stdio command/arguments, not the friendly configured server name. A full URL allowlist entry is not a scheme, port, path, or tenant restriction. Review the actual process invocation or network destination when writing policy.

Persistent MCP approvals are fingerprinted to the stdio command and arguments or to the remote URL. Re-pointing a server invalidates the earlier approval. Approval of one server definition should not be interpreted as approval of a later definition at a different destination.

`blockOnMcpLoad` defaults to `false`, so an agent turn need not wait for all MCP servers to load. Do not assume every configured tool was available at the start of every turn unless the deployment explicitly requires and verifies that behavior.

## Hooks

Current hook configuration can come from:

```text
~/.factory/hooks.json
.factory/hooks.json
org-managed settings
```

Legacy `.factory/hooks/hooks.json` still loads. If `hooks.json` is absent, Droid reads the `hooks` key from the matching `settings.json`.

Documented events are:

```text
PreToolUse
PostToolUse
UserPromptSubmit
Notification
Stop
SubagentStop
PreCompact
SessionStart
SessionEnd
```

Hook commands receive JSON on stdin. Exit behavior is:

| Exit | Effect |
|---|---|
| `0` | Success |
| `2` | Blocking or corrective feedback where supported |
| Other nonzero | Non-blocking failure |

`PreToolUse` can return `allow`, `deny`, `ask`, and `updatedInput`. `Stop` and `SubagentStop` can prevent stopping; `SessionEnd` cannot block termination. These outcomes are event-specific, not a promise that every hook can veto every runtime action.

Hooks are snapshotted at startup. External modifications generate a warning and should be reviewed in `/hooks`; do not assume an edit silently rewrote the active snapshot. Organization-managed hooks always load unless hooks are globally disabled. `hooksDisabled` defaults to `false`.

Hook commands run with the local environment and credentials. Use reviewed scripts and absolute paths, and treat repository-provided hooks as executable code rather than inert lint configuration. A hook can enforce useful deterministic checks, but a non-blocking failure or unsupported event outcome is not a complete security boundary.

## Plugins

A native Factory plugin can use this layout:

```text
my-plugin/
  .factory-plugin/plugin.json
  commands/
  skills/
  droids/
  output-styles/
  hooks/hooks.json
  mcp.json
```

Components remain at the plugin root, not inside `.factory-plugin/`. Compatible Claude Code plugin layouts are translated during cache copy.

Installed identity comes from the marketplace entry plus the registered marketplace name; Droid does not require or read identity fields inside `plugin.json` for that purpose. Enablement is stored in `enabledPlugins`. There is no `droid plugin enable` or `droid plugin disable` command.

For npm-distributed plugins, installation uses:

```text
npm install --ignore-scripts --no-save --no-audit --no-fund
```

Packages therefore need to ship ready-to-use files rather than relying on lifecycle scripts to generate them. Suppressing npm lifecycle scripts does **not** make a plugin safe: its hooks, MCP servers, executable commands, skill scripts, and other content can still execute or grant access after installation.

Organization plugin and marketplace controls can restrict what is available, but they do not turn third-party extension content into a sandboxed artifact. Review source, install location, update channel, executable components, credentials, and network access before enabling a plugin.

## Connectors are a separate integration surface

Connectors use Factory’s managed remote connector service. No local connector process runs. Connections are scoped to the Factory user and active organization, and tools load through a deferred catalog.

Risk defaults are:

| Tool type | Default risk |
|---|---|
| Read-only | Low |
| Write, destructive, or unknown | High |

Connector approvals are action-specific and do not become persistent MCP permissions. Enterprise availability is server-enforced.

Connecting an app does not install the corresponding Delegation app. Connector access supplies tools; a Delegation supplies an application entry point and execution workflow. Evaluate their permissions and identity separately.

## Trust comparison

| Extension | Primary capability | Important boundary |
|---|---|---|
| MCP server | Exposes local or remote tools | Stdio code executes locally; remote endpoints receive tool traffic; approval and filters are not process isolation |
| Hook | Runs deterministic lifecycle commands | Runs with local environment and credentials; only documented outcomes block |
| Plugin | Packages commands, skills, droids, styles, hooks, and MCP | Install-script suppression does not neutralize packaged executable behavior |
| Connector | Factory-managed remote app tools | Separate user/org-scoped connection and action approvals; not a local MCP server |
| Credential helper | Mints provider credentials on demand | Executes a shell command and must be treated as privileged configuration |

## Sources

- [MCP](https://docs.factory.com/harness/mcp)
- [Hooks](https://docs.factory.com/harness/hooks)
- [Plugins](https://docs.factory.com/harness/plugins)
- [Connectors](https://docs.factory.com/harness/connectors)
- [Droid CLI settings](https://docs.factory.com/droid-cli/settings)
- [Enterprise Controls and managed settings](https://docs.factory.com/enterprise/hierarchical-settings-and-org-control)
- [Custom Models (BYOK)](https://docs.factory.com/model-independence/byok)
