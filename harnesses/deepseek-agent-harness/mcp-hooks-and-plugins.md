# MCP, hooks, and plugins

**Verified:** 2026-10-29

## MCP

`@deepseek-ai/dsh-mcp-client` connects external MCP tools and resources to the harness. No MCP server is enabled by default. Configure one plugin row per server in the applicable profile/home patch or CLI overlay.

```yaml
- id: mcp-local
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: local
    transport: stdio
    command: npx
    args: ['-y', 'example-mcp-server']
    env:
      MCP_TOKEN: !!js process.env.MCP_TOKEN

- id: mcp-remote
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: remote
    transport: streamable-http
    url: https://example.com/mcp
    headers:
      Authorization: !!js '`Bearer ${process.env.MCP_TOKEN}`'
```

Supported transports are `stdio` and `streamable-http`. `serverName` is required, must be unique in its registration scope, and uses `[A-Za-z0-9_-]{1,32}`. Model-facing tools are named:

```text
mcp__<serverName>__<rawToolName>
```

The CLI ships the MCP client dependency for patch layers, but a server command is trusted executable code outside the agent sandbox. Review the server, transport, credentials, exposed tools, and network destinations before enabling it.

### Failure and recovery defaults

| Field | Default |
|---|---|
| `toolCallTimeoutMs` | 60,000 ms |
| `maxInstructionBytes` | 32,768 bytes |
| `failOnStartupError` | `false` |
| `reconnect.enabled` | `true` |
| `reconnect.initialDelayMs` | 500 ms |
| `reconnect.maxDelayMs` | 30,000 ms |
| `reconnect.maxAttempts` | 10 |

With `failOnStartupError: false`, initial connection failure can leave the harness running without that server’s tools. During an outage, stale tool definitions can remain listed while calls fail. After the reconnect budget is exhausted, tools are removed until configuration reload or restart.

Stdio children start from a scrubbed environment that drops ambient names matching `KEY|PASSWORD|SECRET|TOKEN` and ambient `DSH_*` names; explicit `env` entries are then merged on top. This reduces accidental inheritance but does not make the server safe.

MCP resources are read on demand. Resource subscriptions and MCP prompt templates are unsupported; task-required MCP tools are rejected at call time.

## Experimental hook bridges

Claude Code and Codex hook support is provided by **experimental** compatibility bridges. These are not a stable, fully compatible replacement for either harness’s hook system, and they are not enabled merely because a hook file exists.

The shared protocol runs only command hooks. `http`, `mcp_tool`, `prompt`, and `agent` handlers are skipped with a warning.

A supported hook can block a prompt or tool call, attach context, or request approval where the dialect supports it. However:

- exit code `2` blocks with stderr as the reason;
- other exit codes and process failures are non-blocking;
- unreadable or invalid configuration logs a warning, runs no hooks, and still starts the agent;
- `updatedInput` is parsed but not honored;
- `{"continue": false}` is recorded but does not halt the overall run.

The Claude Code bridge supports only a documented subset of events and fields. It reads one process-level `configPath` once at load rather than implementing Claude Code’s complete layered discovery and live reload. Review the bridge’s limitations before migrating a policy.

**Security consequence:** do not make one of these experimental hooks the sole enforcement mechanism for a required control. Use a native plugin or an external enforcement boundary with explicit fail-closed behavior.

## Native Cordis plugins

A native plugin is commonly a TypeScript module exporting `apply(ctx)`:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello-plugin'
export const inject = ['tools']

export function apply(ctx: Context) {
  // Register capabilities through ctx.
}
```

`inject` declares required services. Registrations made through `ctx` are effect-scoped and cleaned up on unload; resources needing explicit teardown use `ctx.effect()` and return a disposer. Plugin `Config` schemas validate configuration at load, and invalid self-contained configuration should fail loudly.

A local overlay can insert a plugin by absolute module path:

```yaml
- insert:
    - id: hello
      name: '/absolute/path/to/plugin.ts'
```

## Installable bundles

Manage profile packages through:

```sh
dsh plugin --profile <name> add <package-or-git-spec>
dsh plugin --profile <name> remove <package>
```

The command forwards package operations to `pnpm` in the profile directory. A dependency declaring a bundle layer joins `dsh.profile.bundles`:

```json
{
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

Restart after changing bundle membership. In-box bundles resolve from the running installation first; out-of-tree bundles resolve from the profile’s package environment. Installation and startup enforce declared DSH compatibility ranges, with explicit exact-version exemptions available for acknowledged exceptions.

Plugins can execute code, access services, import native dependencies, and change runtime behavior. `dsh-plugin` repository topics and appearance in a plugin index are discovery aids, not security certification. Review source, dependencies, install scripts, requested services, configuration, and credentials before installation.

## Sources

- [MCP client](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/mcp/mcp-client/README.md)
- [Hook protocol](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/experimental/hook-protocol/README.md)
- [Claude Code hook bridge](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/experimental/hooks-claude-code/README.md)
- [Experimental status](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/experimental/README.md)
- [First plugin](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/index.md)
- [Plugin configuration](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/config.md)
- [CLI plugin management](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
