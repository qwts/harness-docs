# Claude Code — Settings Hierarchy & Enterprise Controls (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

## Settings precedence (five-tier hierarchy)

✅ TRUE — The source draft's Table 6 matches official docs. From highest to lowest:

| Level | Source | Authority |
|---|---|---|
| 1 (highest) | **Managed settings** (`managed-settings.json`, deployed via MDM/Device Management) | Cannot be overridden locally; enforces org policy |
| 2 | **Command-line flags** (e.g. `--model`, `--settings <file-or-json>`) | Single-session overrides |
| 3 | **Project local settings** (`.claude/settings.local.json`) | Private, untracked, per-project experiments |
| 4 | **Shared project settings** (`.claude/settings.json`) | Baseline committed to version control |
| 5 (lowest) | **User settings** (`~/.claude/settings.json`) | Global fallback defaults |

Notes confirmed beyond the draft:
- A key set at a higher tier overrides the same key below it; a key left out keeps its lower-tier value. ✅ TRUE
- A few keys honor the *restrictive* value from any file (e.g. `disableClaudeAiConnectors`), and some managed-source keys merge as a union rather than winner-takes-all (e.g. `env`, `allowAllClaudeAiMcps`). ⚠️ Draft omitted this nuance.
- `permissions.defaultMode` values `auto`/`bypassPermissions` do not take effect from project or local settings — set them in user or managed settings, or pass `--permission-mode`. ⚠️ Draft omitted.
- Run `/status` in-session: the **Setting sources** line lists every settings file loaded and which managed source applies.

## Managed settings locations

| OS | Path |
|---|---|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux / WSL | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-settings.json` (the old `C:\ProgramData\ClaudeCode\` fallback was removed in v2.1.75) |

## Enterprise MCP control: managed-mcp.json

✅ TRUE — the draft's "Managed Scope" row is essentially right:

- `managed-mcp.json` provides **exclusive control**: when deployed and parseable, users cannot add their own servers — `claude mcp add` fails with *"Cannot add MCP server: enterprise MCP configuration is active and has exclusive control over MCP servers"*, and `--mcp-config` makes a workstation CLI exit at startup.
- `claude mcp list` shows only managed servers (plus any provided via the `managedMcpServers` settings key).
- Deploy an empty server map to block every server.
- `allowedMcpServers` / `deniedMcpServers` / `allowManagedMcpServersOnly` in `managed-settings.json` provide allow/denylist control; denylist beats allowlist.
- ⚠️ PARTIAL nuance on `allowAllClaudeAiMcps`: it does **not** unsuppress user-defined/local servers. It allows **claude.ai cloud connectors** to load *alongside* a deployed `managed-mcp.json`. The draft conflated these.

## Environment variable resolution

| Claim | Verdict |
|---|---|
| Desktop inherits GUI-session env, ignoring shell profiles → inject secrets into the config `env` block | ✅ TRUE |
| CLI inherits the active shell's environment | ✅ TRUE |
| CLI settings files can set/override env via the `env` key; these supersede inherited shell variables | ✅ TRUE — the `env` key in settings applies to every session and subprocess it spawns; under managed settings, `env` merges per-key across admin sources rather than winner-takes-all |

## Where credentials should go (agent guidance)

- **Never commit secrets** to project scope: a `.mcp.json` `env` block lands in git history. Use ${VAR` references so the secret stays in the shell/user environment, or keep credentialed servers at local/user scope.
- Credentials for MCP OAuth are stored per endpoint by Claude Code; `claude mcp remove` on a remote server also deletes its stored OAuth tokens and client registration.
- Claude Code redacts credential-like text from connection error output and never prints expanded URLs that may carry secrets.
