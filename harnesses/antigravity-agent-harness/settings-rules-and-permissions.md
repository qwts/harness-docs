# Settings, rules, and permissions

**Verified:** 2026-10-29

## App settings architecture

Antigravity App separates global settings from project settings. Global settings cover account, default permissions, appearance, browser integration, model usage, and customizations. Project settings cover project folders, agent behavior, project permissions, and project customizations.

Projects can inherit the global permission preset or override it. Standalone conversations run in a local scratch directory and have their own settings in addition to inheriting global permissions; they are not ordinary folder-backed projects.

## App permission presets

| Preset | Terminal sandbox | Commands | File access | MCP and web |
|---|---|---|---|---|
| Request Review | Disabled | Ask | Workspace only | Ask |
| Default | Enabled | Allow in sandbox; ask outside | Workspace + temp | Ask |
| Auto (Preview) | Enabled | Allow in sandbox; automatically assess outside | Workspace + temp | Ask |
| Turbo | Disabled | Allowed, unrestricted | Full filesystem | Allowed |

Allow, Ask, and Deny rules layer over the preset and take precedence over its fallback behavior. Under Default and Auto, sandboxed commands have workspace/temp access and no network by default. File and URL grants can expand sandbox filesystem and network reach.

Permission resources use `action(target)`:

```text
read_file(path)
write_file(path)
read_url(domain)
execute_url(domain)
command(prefix)
command(regex:pattern)
mcp(server/tool)
mcp(server/*)
```

`write_file` implies read on the same target; denying read also blocks write. `read_url` covers web reading, browser navigation, and sandbox network allowlisting, while `execute_url` separately covers browser actuation.

Command prefix matching has safeguards. Constructs that can conceal execution, or commands that cannot be parsed cleanly, require a full-line match to run without prompting. Standard pipelines and ordinary `&&`/`||`/`;` composition can still prefix-match.

The previous permission system included `unsandboxed(...)`; current presets use `command(...)` for execution both inside and outside the sandbox. Do not mix the two schema generations.

## CLI settings and permissions

CLI persistent settings are stored at:

```text
~/.gemini/antigravity-cli/settings.json
```

The CLI sparsely persists non-default values, preserves unknown keys, and writes the file with user-only permissions. Hooks, MCP, keybindings, and plugins have separate files.

CLI controls are not just the App preset names serialized into JSON:

- `agentMode`: `default`, `accept-edits`, `auto`, or `plan`;
- `toolPermission`: `request-review`, `proceed-in-sandbox`, `strict`, or `always-proceed`;
- `enableTerminalSandbox`;
- `allowNonWorkspaceAccess`;
- `permissions.allow`, `permissions.ask`, `permissions.deny`.

CLI `request-review` is the default tool-permission mode. CLI terminal sandbox is off by default, while `agentMode: auto` enables it automatically. Deny rules take precedence over Ask, and Ask over Allow. Command-line flags override settings only for the current session.

## Rule discovery and precedence

Rules are cumulative, not replacement-based. Antigravity combines global, workspace, and directory rules; more specific directory rules take priority when instructions conflict.

Directory-scoped sources include:

```text
<dir>/AGENTS.md
<dir>/GEMINI.md
<dir>/.agents/AGENTS.md
<dir>/.agents/GEMINI.md
<dir>/.agents/rules/*.md
```

Legacy `.agent/rules/*.md` is also recognized. Global sources include standalone files under `~/.gemini/` and `~/.gemini/config/`, plus modular `~/.gemini/config/rules/*.md`.

`AGENTS.md` and `GEMINI.md` are plain Markdown and always active for their scope. Every modular rule under `rules/` requires frontmatter with a `trigger`:

| Trigger | Activation |
|---|---|
| `always_on` | Full content active in scope |
| `model_decision` | Path/description indexed; agent reads full rule when relevant |
| `glob` | Activates for matching files; requires `globs` or `glob` |
| `manual` | Loads only after explicit `@` mention |

`description` is required for `model_decision`. Rule files support `@[label](path)` content includes and `@filename` canonical file references; these are different operations.

Limits are 24,000 bytes per file after includes and a 20,000-token aggregate budget for global and `always_on` rules. Oversized aggregate content can be demoted from inline text to on-demand pointers.

## Sources

- [Settings](https://antigravity.google/docs/settings/)
- [Permissions](https://antigravity.google/docs/permissions/)
- [Rules](https://antigravity.google/docs/rules/)
- [CLI settings](https://antigravity.google/docs/cli/settings/)
