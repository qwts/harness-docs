# CLI, cloud, and enterprise surfaces

**Verified:** 2026-10-29

## Cursor CLI

The executable is:

```text
agent
```

The CLI supports interactive Plan and Ask modes, non-interactive `-p` / `--print` execution, structured output, and Agent Client Protocol through:

```sh
agent acp
```

Do not confuse Cursor's `agent` executable with a generic shell command of the same name in an unrelated environment. Headless output and ACP are integration surfaces; they do not change the underlying need to configure permissions and MCP for the actual runtime.

## Cloud environment resolution

For Cloud Agents, environment selection resolves in this order:

```text
personal saved environment
> team saved environment
> repository .cursor/environment.json (single-repository runs)
```

A saved environment reads the repository file only when its configured location points to that file. Merely committing `.cursor/environment.json` does not override a higher-priority saved environment.

Self-Hosted Machines do not read `.cursor/environment.json`. Their host setup must be handled through the self-hosted configuration path rather than assuming repository-file parity.

## Cloud Build lifecycle

The repository environment can define lifecycle commands:

| Field | Execution phase |
|---|---|
| `install` | During the Build |
| `start` | At agent start |
| `terminals` | At agent start for terminal setup |

Builds preserve disk state. They do not preserve running processes or exported shell state. Long-running services therefore belong in an appropriate start phase, not only in `install`.

A failed new Build does not replace the last successful active Build. This protects availability but can also make an agent run on older environment state than the latest repository changes suggest. Verify which Build is active when debugging environment drift.

## Enterprise and team boundaries

Team administrator policy outranks user/workspace permission files and IDE settings. Team Rules likewise outrank Project and User Rules. Cloud MCP, cloud hooks, saved environments, and self-hosted machine setup are operational configuration, not side effects of local IDE preferences.

❓ **UNVERIFIED:** This document does not claim that every IDE-only feature is available identically in CLI, Cloud Agents, automations, or Self-Hosted Machines. Use the surface-specific documentation for deployment.

## Sources

- [Using Cursor CLI](https://cursor.com/docs/cli/using)
- [Cloud Agent setup](https://cursor.com/docs/cloud-agent/setup)
- [Cloud Builds](https://cursor.com/docs/cloud-agent/builds)
- [Permissions reference](https://cursor.com/docs/reference/permissions)
- [Rules](https://cursor.com/docs/rules)
