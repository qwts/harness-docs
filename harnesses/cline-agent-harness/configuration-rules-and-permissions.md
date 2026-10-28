# Configuration, rules, and permissions

**Verified:** 2026-10-29

## Configuration and state roots

Cline’s shared global configuration root is `~/.cline/`; application state lives below `~/.cline/data/`. Project configuration uses `.cline/`, while `.clinerules/` remains relevant for rules and for the VS Code extension’s hook layout.

```text
~/.cline/
  data/
    settings/
      providers.json
      global-settings.json
      cline_mcp_settings.json
  rules/
  workflows/
  hooks/
  skills/
  agents/
  plugins/
  cron/

.cline/
  rules/
  workflows/
  skills/
  hooks/
  agents/
  plugins/
```

Do not put provider credentials in committed project configuration. Keep secrets in the appropriate local settings, secret store, or environment.

### Location overrides

| Setting | Effect |
|---|---|
| `CLINE_DIR` | Replaces the default `~/.cline/` root, like `--config` |
| `CLINE_DATA_DIR` | Replaces the default `~/.cline/data/` state directory |
| `CLINE_MCP_SETTINGS_PATH` | Overrides the MCP settings file path |
| `CLINE_HOOKS_DIR` | Adds a hook discovery location; `--hooks-dir` is the CLI equivalent |

These overrides matter in automation: a job can isolate configuration and mutable state without writing into the runner user’s normal home-directory state.

## Rule discovery

Cline combines rule sources rather than reducing the workspace to one instruction file.

| Scope | Documented sources |
|---|---|
| Current workspace layout | `.cline/rules/` |
| Legacy/compatibility workspace layout | `.clinerules/` |
| Other workspace instruction files | Root `AGENTS.md`, `.cursorrules`, `.windsurfrules` |
| Global rule directories | `~/.cline/rules/`, `~/Documents/Cline/Rules/`, `~/Cline/Rules/` |
| Global agent instructions | `~/.agents/AGENTS.md` |

Both `.clinerules/` and `.cline/rules/` are searched when present. Workspace rules override global rules on conflict; this is conflict resolution, not a claim that only one scope is loaded.

In a multi-root IDE workspace, `.clinerules/` applies only to the primary workspace. Do not infer that every root contributes an independent rule tree.

## Conditional rules

A rule with no frontmatter is always active. YAML frontmatter can restrict activation with path globs:

```markdown
---
paths:
  - "src/**/*.ts"
  - "tests/**/*.ts"
---

Run the TypeScript checks after changing these files.
```

Activation context can include mentioned paths, open or visible files, edited files, and pending operations.

| Frontmatter condition | Behavior |
|---|---|
| No frontmatter | Always active |
| Matching `paths` glob | Activates for relevant context |
| `paths: []` | Never activates |
| Invalid YAML | Fails open: activates the rule and exposes the raw content |

Fail-open parsing is significant for security-sensitive instructions. A malformed rule is not a reliable deny control; enforce security decisions in code, hooks, or the surrounding runtime.

## Plan, Act, and approval behavior

Plan and Act are operating modes, not interchangeable names for the same permission policy. Plan mode restricts normal editing and some shell commands, but it is not a guarantee that no write can occur. Treat it as a workflow aid rather than a hard security boundary.

Interactive surfaces expose Auto Approve controls for categories of tool activity. CLI defaults require separate attention:

- A prompt run such as `cline "task"` defaults to Act mode.
- `--auto-approve` defaults to `true` for ordinary prompt runs.
- In ACP mode, the auto-approval default is `false`.
- Headless behavior is triggered by `--json`, piped stdin, or redirected stdout.

Before unattended execution, inspect the actual flags and effective approval behavior. Do not carry an interactive IDE approval assumption into a shell pipeline.

## Shell-command policy

Current CLI documentation does **not** provide a shell-command allow/deny list. The documented enforcement approach is a `PreToolUse` hook that inspects `run_commands` calls and cancels disallowed operations.

A hook is executable policy code. Review it, test bypass cases, and run it with least privilege. Pattern matching a command string is not equivalent to parsing every shell construct.

## `.clineignore` status and limits

Built-in `.clineignore` behavior is deprecated. Cline’s replacement example uses a `PreToolUse` guard to block reads, edits, and shell commands involving ignored paths.

A cancellation response can take this form:

```json
{
  "cancel": true,
  "errorMessage": "Blocked read_files: .env matched a .clineignore pattern."
}
```

The documented replacement is not a complete access-control system:

- it does not filter search or file-listing results;
- it is not a full shell parser;
- it does not resolve symlink aliases.

Use filesystem permissions, isolated credentials, sandboxing, and reviewed hooks for sensitive material. A `.clineignore` file alone must not be treated as a secrecy boundary.

## Sources

- [Configuration](https://docs.cline.bot/getting-started/config)
- [Cline rules](https://docs.cline.bot/customization/cline-rules)
- [Auto Approve](https://docs.cline.bot/features/auto-approve)
- [Plan and Act](https://docs.cline.bot/core-workflows/plan-and-act)
- [CLI reference](https://docs.cline.bot/cli/cli-reference)
- [Hooks](https://docs.cline.bot/customization/hooks)
- [`.clineignore`](https://docs.cline.bot/customization/clineignore)
