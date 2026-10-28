# Configuration, rules, and permissions

**Verified:** 2026-10-29 · **Primary surface:** Cursor IDE/local agent

## Project rules

Cursor project rules are Markdown-with-frontmatter files using the `.mdc` extension:

```text
.cursor/rules/**/*.mdc
```

Plain `.md` files in that directory are not project-rule files. Rule activation is controlled by frontmatter:

| Field | Role |
|---|---|
| `alwaysApply` | Apply the rule regardless of file matching |
| `description` | Make the rule available for relevance-based selection |
| `globs` | Apply to matching files |

Applicable rules are merged. When rules conflict, Cursor documents this scope precedence:

```text
Team Rules > Project Rules > User Rules
```

This is not a claim that only one rule file is loaded. Multiple applicable rules can coexist.

## `AGENTS.md`

`AGENTS.md` is the simpler plain-Markdown alternative for repository instructions. Nested `AGENTS.md` behavior is supported, allowing more local instructions near the files they govern. Do not rename an `.mdc` rule to `.md` and assume equivalent activation metadata will still be interpreted.

## Permission files

Cursor reads JSON permission configuration at:

```text
~/.cursor/permissions.json
<workspace>/.cursor/permissions.json
```

Arrays from the user and workspace files are concatenated. Source-level precedence is:

```text
team administrator policy > permissions.json > IDE settings
```

That ordering is distinct from the rule-instruction precedence above. “Project overrides user” is not a complete description of Cursor permissions.

## Run Modes

| Mode | Behavior |
|---|---|
| Auto-review | Check allowlisted calls first, sandbox shell commands where possible, and send remaining eligible calls to an automatic reviewer/classifier |
| Allowlist | Permit configured calls and require review for others |
| Run Everything | Bypass ordinary command-by-command review; highest-risk mode |

Auto-review is a best-effort guardrail, not a hard security boundary. Sandboxing is used where possible rather than guaranteed for every operation. Command classification does not make arbitrary shell safe, and explicit organizational policy remains a separate layer.

## Practical evaluation order

When diagnosing why a command ran or prompted, check in this order:

1. Team/administrator restrictions.
2. User and workspace `permissions.json` entries.
3. IDE permission settings.
4. Selected Run Mode.
5. Whether sandboxing was actually available for that call.

Do not infer the effective result from a single allowlist entry without checking higher-priority policy and deny behavior.

## Sources

- [Rules](https://cursor.com/docs/rules)
- [Permissions reference](https://cursor.com/docs/reference/permissions)
- [Agent security](https://cursor.com/docs/agent/security)
- [Run Modes](https://cursor.com/docs/agent/security/run-modes)
