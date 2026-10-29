# Instructions, permissions, and sandbox

**Verified:** 2026-10-29

## Instruction discovery

Base-backed profiles load user-global and project instructions. Standalone `sdk-minimal` does not include local instruction discovery.

### Global and project chains

The user-global chain is:

```text
$DSH_HOME/AGENTS.md
$DSH_AGENTS_HOME/AGENTS.md
```

The project chain loads applicable candidates from the project root down to the session working directory, broad to specific. `.git` marks the project root by default.

| Candidate | Default role |
|---|---|
| `AGENTS.md` | Shared project instructions |
| `CLAUDE.md` | Compatible project instructions |
| `AGENTS.local.md` | Additive local overlay |
| `CLAUDE.local.md` | Additive local overlay |

Content-identical candidates within one candidate group render once after trimming. Changing the session working directory reloads the applicable project chain before the next request.

### Budgets and refresh

The default budget is **65,536 bytes per chain**, not one budget shared by both chains. A message containing both chains can therefore reach twice that amount. Within a chain, broader files are omitted before the most specific file is truncated; content is bounded, not model-summarized.

Refresh is not continuous file watching:

- the first request loads the baseline;
- successful structured `read`, `write`, or `edit` operations discover newly relevant nested files and make later changes visible;
- resume reconciles the visible baseline.

A shell command that changes directories does not trigger nested instruction discovery. Lowercase filenames, `.claude/rules/`, and `@path` imports are not interpreted by the default candidate semantics.

Symlinked instruction files are followed, including targets outside the repository. Treat cloned instruction files as lower-authority, untrusted guidance and confine filesystem access when loading untrusted repositories.

## Permission presets bundle two controls

The standard presets combine sandbox mode and approval policy:

| Preset | Sandbox | Approval |
|---|---|---|
| `workspace-write` | `workspace-write` | `ask` |
| `danger-full-access` | `danger-full-access` | `never` |

Base-backed `web`, `headless`, and `sdk` profiles default new sessions to `workspace-write`. `DSH_PERMISSION_MODE` changes the process fallback. Stored Web permission settings affect later sessions, not an already-open session; resumed sessions preserve effective permissions.

A `/permission` command can report or switch presets where a command registry is composed. An unmatched sandbox/approval combination appears as derived `custom`; users can leave that state but cannot select `custom` as a named preset.

**Surface exception:** standalone `sdk-minimal` pins `danger-full-access` and mounts no approval or permission-settings service. Do not apply the base-profile default to it.

## File-effect policy

The shared sandbox policy resolves each confined call in this order:

1. an approved explicit per-call mode;
2. the session’s recorded sandbox mode;
3. the deployment default.

The plugin’s own fail-safe configuration default is `read-only`; a deployment can explicitly select `workspace-write`. The session’s immutable working directory is its normal workspace root.

`additionalWritableRoots` grants physical absolute directories under `workspace-write`. It does not grant them under other modes. Missing paths, files, and symlinked roots are rejected. For linked Git worktrees, granting shared `.git` metadata also grants the ability to modify hooks and configuration that later host Git operations may execute; use that grant only for trusted tasks.

### What the modes do not cover

The sandbox vocabulary governs **file effects**, not every resource:

- base-profile `workspace-write` restricts Bash and filesystem mutations to the workspace and platform temporary roots;
- reads are not confined by that mode;
- network access is not confined by that mode;
- process visibility depends on the sandbox backend.

Enabled web-fetch calls run in every sandbox and approval mode without per-call confirmation and can reach loopback and private-network hosts. A file sandbox therefore is not an SSRF or egress-control mechanism.

## Safety boundary

The official safety notice states that sandboxing, approval prompts, and permission controls reduce risk but do not guarantee isolation or prevent damage. DeepSeek Harness has not undergone a security audit and should not be the sole security control for untrusted workloads.

Use a disposable VM or container, least-privilege credentials, backups, reviewed plugins and commands, and explicit network controls where the workload requires them.

## Sources

- [Agent instructions](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/context/agent-instructions/README.md)
- [Permission presets](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/interaction/permission-presets/README.md)
- [Sandbox policy](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/sandbox/sandbox-policy/README.md)
- [CLI behavior reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
- [Python SDK guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/python-sdk.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
