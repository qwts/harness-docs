# Configuration Files, Settings & Permissions (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against `cli/reference/configuration/*`, `cli/reference/permissions`, `cli/enterprise/*`, and `cli/sandbox` (docs.devin.ai). All claims are ✅ TRUE unless marked.

## Where configuration lives

| Layer | Location | Notes |
|---|---|---|
| User config | `~/.config/devin/config.json` — Windows: `%APPDATA%\devin\config.json` | Personal defaults: model, theme, global permissions, `attribution`, user-only `sandbox` |
| Project config | `.devin/config.json` | Committed to VCS. **Restricted key set**: `permissions`, `read_config_from`, `hooks`, plus repo-level plugin governance lists `requiredPlugins` / `optionalPlugins` / `forbiddenPlugins`. |
| Project-local overrides | `.devin/config.local.json` | Personal overrides + secrets; auto-excluded via `.git/info/exclude` |
| MCP servers (user/project/local) | `~/.config/devin/mcp_config.json`, `.devin/mcp_config.json`, `.devin/mcp_config.local.json` | Dedicated files since v3000.3 — see [mcp-and-extensibility.md](mcp-and-extensibility.md) |
| Machine policy | `system.json` in an admin-writable system directory (MDM-distributed) | Pins enterprise host/account, forces outbound proxy; user cannot override |
| Team/org settings | Server-side (Devin app → Settings → Enterprise, or legacy windsurf.com/team dashboards) | Applied on sign-in; highest precedence |

✅ TRUE — all harness-level config is **JSON**, not TOML. (Contrast: Codex uses TOML; Claude uses JSON. Devin follows the JSON convention.)

## Permission rule model

Tool calls are matched against `permissions.allow` / `permissions.ask` / `permissions.deny` lists of scoped patterns:

```json
{
  "permissions": {
    "allow": ["Read(**)", "Exec(git status)", "mcp__github__*"],
    "ask":   ["Exec(npm run build)"],
    "deny":  ["Exec(sudo)", "mcp__github__delete_repo"]
  }
}
```

- `Read(...)` / `Write(...)` take glob scopes; `Exec(...)` takes command prefixes; `mcp__server__tool` / `mcp__server__*` / `mcp__*` pattern MCP tools.
- **Resolution at one level**: deny → ask → allow → default prompt. **A deny rule always wins** — no allow rule can override a matching deny regardless of specificity.
- `ask` prompts unless an equally-or-more-specific command/MCP allow rule also matches.

## Precedence across sources (highest first)

1. Organization/team settings (enterprise) — org deny/ask cannot be overridden by any file
2. Session-level grants (interactive approvals)
3. `.devin/config.local.json`
4. `.devin/config.json`
5. `~/.config/devin/config.json` (`%APPDATA%` on Windows)

⚠️ Note the inversion vs. intuition: **project-local `.local` file outranks committed project config**, and both outrank user config.

## Permission modes (agent-facing)

`Shift+Tab` cycles modes; `--permission-mode <mode>` selects at launch; `/mode <name>` or dedicated slash commands switch mid-session.

| Mode | Behavior |
|---|---|
| Normal (default) | Auto-approves read-only tools in cwd; prompts for writes/exec |
| Accept Edits | Auto-approves file edits in workspace; prompts for shell/other actions |
| Smart | Accept Edits + a fast model auto-judges shell/web/MCP calls; high-risk categories (package installs, mutating git, `rm`, `sudo`, destructive cloud ops, sensitive files) always prompt. Gradual rollout — may be absent. |
| Bypass | Auto-approves everything. Aliases `/yolo`, `/dangerous`. **Never** overrides org-level deny/ask. |
| Autonomous | Only available with `--sandbox` (and then the only mode): prompts by capability, not command; Write scopes + Read denies enforced at OS level; network connects prompt |

Agent-side modes: **Normal**, **Plan** (`/plan`, Alt+P), **Ask** (`/ask`) — orthogonal to permission mode.

## `system.json` — machine policy

Optional, additive, admin-only file distributed via MDM. Uses:
- Pin auth to enterprise Devin host/account (login menu skipped; foreign accounts rejected)
- Force outbound HTTP proxy for CLI + updater — user config `proxy` must then be removed before CLI starts
Absent = unmanaged behavior. ✅ TRUE

## Team settings (server-side)

Applied on sign-in; cover Devin CLI **and** Devin Desktop (same store): MCP enable/allowlist, MCP registry, permission rules (highest precedence), default model + model allowlist, CLI version constraint (semver, e.g. `>=1.2.0 <2.0.0`), sandbox enforcement, web search, extension policy/marketplace URL, agent hooks. Devin CLI access itself is gated by a custom RBAC role permission **Use Devin CLI**.

## Sandbox

- `--sandbox` activates OS-level **filesystem** containment (writable roots + Read denies); **fails closed** — if sandboxing can't be established the CLI refuses to start. Network containment is **optional** and only activates when domain filtering is configured (next bullet); with empty domain lists, child-process network access is not restricted.
- Writable roots derive from granted `Write(...)` scopes + workspace dirs; mid-session grants expand it dynamically.
- Optional domain filtering (user config `sandbox` section): `allowed_domains` (non-empty = allowlist mode), `denied_domains` (deny wins), `network_mode` `full|limited` (limited = GET/HEAD/OPTIONS only). Domain syntax: `example.com` exact, `*.example.com` subdomains only, `**.example.com` apex + subdomains.
- Enterprise: allowlists are authoritative (replace local), denylists are additive (merged).
- ⚠️ Docs flag sandbox **network filtering as currently unstable**.
