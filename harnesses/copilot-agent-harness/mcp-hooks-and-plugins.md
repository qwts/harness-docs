# MCP, hooks, and plugins

**Verified:** 2026-10-29 · **Primary surface:** Copilot CLI

## MCP configuration

User configuration:

```text
~/.copilot/mcp-config.json
```

Project discovery walks from the working directory to the repository root and reads:

```text
.mcp.json
.github/mcp.json
```

For duplicate server names:

1. Project definitions beat user definitions.
2. A file closer to the working directory beats one farther away.
3. In the same directory, `.mcp.json` beats `.github/mcp.json`.

Project files accept either an `mcpServers` wrapper or a bare server-name map. They load only after folder trust. In non-interactive prompt mode, an untrusted directory skips project MCP by default; `GITHUB_COPILOT_PROMPT_MODE_WORKSPACE_MCP=true` deliberately overrides that skip.

`.vscode/mcp.json` is not a Copilot CLI configuration source. Its top-level `servers` shape is not the CLI's documented schema.

The CLI supports stdio, Streamable HTTP, and legacy SSE. The built-in GitHub MCP server needs no additional CLI configuration. Tool allowlists, environment variables, HTTP headers, enable/disable operations, and registry/organization policy are separate controls from approval permissions.

## Hooks

Hook files are JSON with `version: 1`:

```text
Repository: .github/hooks/*.json
User:       ~/.copilot/hooks/*.json
```

Documented events include `sessionStart`, `sessionEnd`, `userPromptSubmitted`, `preToolUse`, `postToolUse`, `errorOccurred`, and `agentStop`. Commands can provide separate `bash` and `powershell` implementations, plus `cwd`, `env`, and `timeoutSec`. The default timeout is 30 seconds.

Hook configuration is loaded at CLI startup. Scripts receive JSON on stdin; troubleshooting output belongs on stderr, and structured stdout must be valid single-line JSON where the event contract requires it.

## Plugin formats

Copilot CLI supports two related but different layouts.

### Agent Plugins 1.0

The manifest is a root `plugin.json` with the canonical Agent Plugins `$schema`. Portable components have fixed locations:

```text
plugin.json
mcp.json
skills/<skill-name>/SKILL.md
com.github.copilot/agents/
com.github.copilot/hooks/hooks.json
```

The portable format covers skills and MCP; Copilot-specific components live under the `com.github.copilot` namespace. For stdio MCP, Copilot expands `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in supported `args`, `env`, and `cwd` fields.

### Legacy Copilot plugins

A manifest without the Agent Plugins `$schema` uses Copilot's legacy behavior. Component paths can be configured in `plugin.json`, including `agents`, `skills`, `hooks`, and `mcpServers`.

Installed plugin components are cached. Reinstall a local plugin after changing it. Repository defaults, local member choices, and enterprise managed plugin policy are distinct layers; enterprise requirements or blocks take precedence.

## Sources

- [Adding MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
- [Using hooks](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks)
- [Creating plugins](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)
- [Copilot CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference)
