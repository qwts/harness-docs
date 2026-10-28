# Permissions and runtime

**Verified:** 2026-10-29 · **Surface:** Muse Code local runtime

## Permission profiles

The default permission profile for a new session is **Auto-review**. Other documented profiles are:

- **Ask me**;
- **Unrestricted**;
- **Read-only**.

Approval modes are `on-request`, `untrusted`, and `never`. Profile choice and approval mode are related controls, but neither should be described as a replacement for sandboxing or path protection.

## Compound commands

Muse Code reviews compound shell commands stage by stage. Rejecting a stage prevents the entire compound command from running. This is stronger than treating each `&&`, pipe, or chain element as an independently approved execution after earlier stages have already run.

## Sandbox

The sandbox is enabled by default and fails closed:

| Platform | Mechanism |
|---|---|
| macOS | Seatbelt |
| Linux | Bundled bubblewrap helper |
| Windows | Windows sandbox; first initialization can require UAC |

The workspace and temporary directory are writable, but these paths remain read-only inside a writable workspace:

```text
.git
.muse
.agents
```

This protects repository metadata and agent configuration from ordinary writes even while project files are editable.

## Network modes

| Mode | Meaning |
|---|---|
| `proxy-only` | Default; network access goes through the controlled proxy path |
| `restricted` | Restricted network policy |
| `unrestricted` | Full network egress |

Disabling the sandbox with `--disable-sandbox` has broader effects than its name might suggest:

- command sandboxing is removed;
- file-tool workspace confinement is removed;
- full network egress is forced.

It is therefore a high-risk runtime mode, not a narrow troubleshooting flag for one command.

## Subagent workspaces

Subagents share the lead checkout by default. Worktree isolation is selected per child. Muse Code does not silently fall back to a shared checkout when isolation was requested.

`--subagent-worktree-isolation` is a compatibility flag; it does not mean every child is automatically isolated. Evaluate each subagent's selected workspace mode instead of inferring it from the flag name.

## Sources

- [Permissions](https://dev.meta.ai/docs/muse-code/permissions)
- [Extending Muse Code](https://dev.meta.ai/docs/muse-code/extending)
