# Instructions, permissions, and sandbox

**Verified:** 2026-10-29

## Instruction discovery

`AGENTS.md` is the recommended durable project-instruction file. With a Git root present, Droid searches from the current working directory up to that root. At each level it checks the directory itself and these context directories:

```text
.factory/
.agents/
.agent/
```

Personal instruction directories are:

```text
~/.factory/
~/.agents/
~/.agent/
```

Compatible filenames are:

```text
AGENTS.md
agents.md
Agents.md
CLAUDE.md
Claude.md
```

Use one appropriate file rather than duplicating instructions across every compatible name. `DESIGN.md`, `Design.md`, and `design.md` are loaded separately as design guidance. Reusable procedures belong in skills when their full text should load only when relevant; settings files are configuration, not prose instructions.

Nested project instructions refine root guidance for their directory tree. Droid can discover them dynamically after reading files in a deeper tree; starting at the repository root does not imply that every nested instruction file was loaded initially.

Instruction-loading budgets are:

| Load phase | Limit |
|---|---:|
| Initial AGENTS-style guideline load | 80,000 characters |
| Dynamic Read-path guideline discovery | 40,000 characters |

These are caps, not recommended sizes. Project instructions are executable context for a tool-using agent: review repository-provided instructions and avoid treating them as a security boundary.

## Interaction mode is separate from autonomy

Factory has three interaction modes:

| Mode | Behavior |
|---|---|
| Normal | Executes work with enabled tools; approvals depend on autonomy, command policy, sandbox checks, and organization controls |
| Spec | Read-only investigation and planning; implementation begins after plan approval and return to Normal Mode |
| Mission | An orchestrator coordinates workers, validation, and progress tracking; workers have their own mission settings |

Some CLI and IDE labels combine Normal Mode and autonomy as `Auto (Off)`, `Auto (Low)`, `Auto (Medium)`, or `Auto (High)`. Mission Mode is not simply a stronger Spec Mode.

Spec Mode says Droid should not edit files, change configuration, commit, start services, or write to external systems during planning. It is a workflow control, not a substitute for OS isolation or least-privilege credentials.

## Autonomy Levels

Autonomy Level controls which risk classes can proceed without repeated approval; it does not determine which tools exist.

| Level | What can run without approval |
|---|---|
| Off | Built-in read tools and commands allowed by policy |
| Low | File edits plus low-risk commands and MCP tools |
| Medium | Low plus reversible workspace changes such as installs, builds, and local commits |
| High | High-risk actions unless another safety control stops or asks |

Effective command policy is evaluated before autonomy fallback:

- `allow` skips command approval but does not make the command safe or read-only;
- `ask` still prompts at High;
- `block` rejects the command with no approval path;
- no matching rule falls back to the session’s approval behavior and Autonomy Level.

Sandbox read, write, and network checks can prompt separately. Built-in safeguards can also require approval for recognized destructive operations.

`--skip-permissions-unsafe` skips permission prompts but not effective command blocks. It is not a policy reset and should be reserved for appropriately isolated environments.

Surface defaults matter. Interactive sessions use their configured session default, while `droid exec` is read-only unless an explicit `--auto low|medium|high` or another documented execution option enables mutations. Task-launched subagents inherit parent autonomy by default, can use a separate subagent setting, and remain clamped by organization maximums. Missions require High autonomy or `--skip-permissions-unsafe`.

## Native permission rules

Native rules govern shell-command decisions. Each rule has a stable ID, a decision, an exact ordered prefix, and both positive and negative tests:

```json
{
  "permissionRules": {
    "version": 1,
    "rules": [
      {
        "id": "project/git-push",
        "decision": "ask",
        "reason": "Review remote Git changes before pushing.",
        "match": { "prefix": ["git", "push"] },
        "tests": {
          "match": ["git push origin main"],
          "noMatch": ["git status", "git log --oneline"]
        }
      }
    ]
  }
}
```

Matching uses exact, ordered, case-sensitive argument tokens. `*` is literal, not a wildcard. A rule for `['git', 'push']` does not match `git -C /repo push`.

Validate files and preview decisions with:

```sh
droid rules check
droid rules check --command 'git push origin main' --json
```

Exit `0` means the check ran successfully, even when the previewed decision is `block`.

Eligible rule sources resolve IDs in this order:

```text
Org > Runtime > Folder > Project > User
```

Within a folder, `settings.local.json` precedes `settings.json`. Different IDs accumulate; a higher-priority valid definition claims a repeated ID. After resolution, matching decisions rank `block > ask > allow`. An organization `allow` therefore does not cancel a project `block` under another ID.

Project and folder rules require workspace trust. Plugins and dynamic configuration cannot author native permission rules. A disabled rule still suppresses lower-priority definitions with the same ID; deleting it removes that suppression. An empty rules array does not clear inherited rules.

Local invalid rules are skipped rather than converted into block-all policy. A lower-priority valid definition with the same ID can consequently become effective. Always run the checker instead of treating a malformed file as a safe failure.

### Rule limits

Permission prefixes do not cover every equivalent command spelling, inspect arbitrary script behavior, or confine every tool. A permitted shell program can perform many actions after launch. Use sandboxing, managed hooks, least-privilege credentials, and isolated compute for defense in depth.

## Sandbox

Factory sandboxing provides operating-system-backed filesystem and network controls:

| Setting | Behavior |
|---|---|
| `per-command` | Default mode; confines applicable commands individually |
| `whole-process` | Confines the main Droid process and everything it spawns |

macOS uses Seatbelt. Linux and WSL2 use bubblewrap with seccomp. Network filtering uses an HTTP/SOCKS proxy. If required isolation is unavailable, the affected action is blocked; in whole-process mode Droid refuses to start.

Default policy is:

```text
Reads:    allow all except denyRead
Writes:   deny all except CWD and allowWrite; denyWrite wins
Network:  deny all except Factory and WorkOS domains plus allowedDomains
```

Example:

```json
{
  "sandbox": {
    "enabled": true,
    "mode": "per-command",
    "filesystem": {
      "allowWrite": ["/tmp/build-output"],
      "denyWrite": ["/tmp/build-output/cache/locks"],
      "denyRead": ["~/.aws/credentials"]
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

Organization deny lists union across settings and cannot be removed downstream. Non-interactive sandbox violations are automatically denied.

Sandboxing is still defense in depth, not a universal boundary for untrusted code. Allowed writable paths remain available for persistence or tampering, and allowed network domains remain exfiltration channels. `per-command` also does not confine the main process as `whole-process` does. Use a container or VM with least-privilege credentials for hostile repositories, dependencies, or scripts.

## Sources

- [AGENTS.md](https://docs.factory.com/harness/agents-md)
- [Interaction Modes](https://docs.factory.com/autonomy-and-safety/specification-mode)
- [Autonomy Level](https://docs.factory.com/autonomy-and-safety/auto-run)
- [Permission rules](https://docs.factory.com/autonomy-and-safety/permission-rules)
- [Sandbox](https://docs.factory.com/autonomy-and-safety/sandbox)
- [Enterprise Controls and managed settings](https://docs.factory.com/enterprise/hierarchical-settings-and-org-control)
