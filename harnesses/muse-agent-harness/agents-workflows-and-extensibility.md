# Agents, workflows, and extensibility

**Verified:** 2026-10-29

## Agent-tree capacity

The default agent-tree capacity is eight agents, including the root. It is configurable from 1 to 64. The root-inclusive count matters: a capacity of eight does not mean eight children plus the lead agent.

Subagent workspace behavior is documented separately: shared checkout by default, per-child worktree selection, and no silent fallback from requested isolation to sharing.

## Hooks

Project hooks use:

```text
.muse/hooks.json
```

User hooks are configured in `settings.json`. Managed hooks use `managed_hooks_path`. These are three configuration channels; do not copy a project hook file into the user settings path and expect the same schema or scope.

## MCP

MCP servers are configured through `mcp_servers` in settings. Muse Code supports stdio and Streamable HTTP, with:

- required and optional server startup modes;
- `${VAR}` environment interpolation;
- `MUSE_SESSION_ID`;
- OAuth login through `muse mcp login`.

A required server failing startup is materially different from an optional server failing. When diagnosing missing tools, inspect both the server definition and its startup requirement rather than treating all configured servers as best-effort.

## Workflows

Workflow availability is rollout/build dependent. Saved project workflows live at:

```text
.agents/workflows/*.js
```

Workflow modes are:

- `auto`;
- `explicit`;
- `off`.

A workflow can make up to 1,000 child calls over its lifetime. This is a lifetime budget, not a per-turn recommendation or a guarantee that every build exposes workflows.

❓ **UNVERIFIED:** A given installation exposes workflows unless its build/rollout is checked. The file format and limits documented here do not prove availability on every deployment.

## Sources

- [Extending Muse Code](https://dev.meta.ai/docs/muse-code/extending)
- [Workflows](https://dev.meta.ai/docs/muse-code/workflows)
- [Configuration](https://dev.meta.ai/docs/muse-code/configuration)
