# Corrections Log — Common Misinformation Audit

Unlike the sibling harnesses, this entry was not derived from a source draft. This log audits **claims commonly repeated about Devin CLI / Devin Desktop** (migration guides, blog coverage, and stale Windsurf documentation) against official docs.devin.ai as of 2026-10-09. Only non-✅ items are listed; everything verified is organized into the sibling topic files.

## ❌ False claims (do not rely on)

| # | Claim | Reality |
|---|---|---|
| F1 | "MCP servers go in the `mcpServers` key of `~/.config/devin/config.json` / `.devin/config.json`." | Since **v3000.3 (Local 3.6)** they live in dedicated `mcp_config.json` files (`~/.config/devin/mcp_config.json`, `.devin/mcp_config.json`, `.devin/mcp_config.local.json`; `%APPDATA%` on Windows). `mcpServers` found in main configs is auto-migrated on startup — docs advising edits to config.json describe the pre-v3000.3 layout. |
| F2 | "`.devin/config.json` accepts the same keys as user config (model, theme, attribution)." | Project config accepts only `permissions`, `read_config_from`, `hooks`, and the repo plugin-governance lists `requiredPlugins`/`optionalPlugins`/`forbiddenPlugins`. `agent.model`, `attribution`, `sandbox`, theme are **user-only** keys — put them in `~/.config/devin/config.json`. |
| F3 | "Devin Desktop's MCP config is `~/.codeium/windsurf/mcp_config.json`." | That file is the **editor's MCP discovery** source (stable vs `windsurf-next`), not the agent's. Devin Local/Cortex reads `~/.config/devin/mcp_config.json` (`$XDG_CONFIG_HOME/devin/` honored) or `%AppData%\devin\mcp_config.json`. Two different files, both real. |
| F4 | "Devin config is TOML / YAML." | All Devin CLI/Desktop harness config is **JSON** (`config.json`, `mcp_config.json`, `hooks.v1.json`, `system.json`, plugin `plugin.json`). TOML is Codex's format. |
| F5 | "Write team rules to `.devinrules`." | No such format. Single-file legacy rules = `.windsurfrules` (still read). Devin-native = `.devin/rules/*.md` + `.devin/global_rules.md`. |

## ⚠️ Partially true / stale

| # | Claim | Correction |
|---|---|---|
| P1 | "Devin Desktop runs the Cascade agent." | **Stale.** Cascade was removed in Devin Desktop **v3.9.19 (2026-09-08)**; Devin Local — same harness as Devin CLI, launched via `devin acp` — is the only bundled agent. Cascade memories/workflows are unsupported by Devin Local (migrate via the Migration Wizard → skills). Cascade-specific docs (hooks `pre_mcp_tool_use`, Turbo mode, app deploys) describe a removed surface. |
| P2 | "User config overrides project config." | Reversed for permissions. Precedence (highest first): **org/team → session grants → `.devin/config.local.json` → `.devin/config.json` → user config**. Project layers beat user; org beats everything. |
| P3 | "A specific `allow` beats a broad `deny`." | Never. Deny always wins regardless of specificity; the resolution order is deny → ask → allow → prompt. |
| P4 | "`devin mcp add` writes to user/global scope." | Defaults to **`local`** scope (per-project, private). `-s user` or `-s project` is explicit opt-in — same model as `claude mcp add`. |
| P5 | "Bypass/YOLO mode skips all checks." | Org-level deny/ask rules still apply — **Bypass never overrides team settings**. |
| P6 | "`subagent: true` in a skill respects its `allowed-tools`/`permissions` frontmatter." | Silently ignored — the subagent runs `subagent_general` with all tools. Use `agent:` + a custom profile with `allowed-tools` to constrain. |
| P7 | "Plugin hooks enforce policy." | Plugin `hooks.json` is **best-effort and fails open**: load/run failures don't stop the session. Not a guardrail mechanism. |
| P8 | "`--sandbox` is just isolation." | It also swaps the permission model: Autonomous is the only mode, prompts are capability-based (Write scopes, network connects), and startup **fails closed** if sandboxing is unavailable. Docs additionally flag network filtering as unstable. |
| P9 | "Devin Desktop reads only `.devin/` rules." | Both `.devin/rules/` and `.windsurf/rules/` load together; the precedence applies only to `global_rules.md` (`.devin` wins). Also `AGENTS.md` files: root = always-on, subdirectory = implicit glob `<dir>/**`. |
| P10 | "AGENTS.md is fully injected into context." | Only the **first 16 KiB** auto-loads; remainder needs on-demand read. Critical instructions go first. |
| P11 | "Devin Desktop = the Devin web app." | Distinct surfaces sharing auth + team settings. Desktop is the rebranded Windsurf IDE with the Devin Local harness; the web app drives Devin cloud sessions. `devin desktop` / `/open desktop` bridges CLI → Desktop. |

## ❓ Unverified (no official confirmation)

| # | Claim | Status |
|---|---|---|
| U1 | "Devin CLI is open source." | ❓ No public repo or license statement found in official docs; the binary ships via installers/Homebrew cask. Treat as closed-source unless Cognition states otherwise. |
| U2 | "Devin Desktop works fully offline / air-gapped." | ⚠️ Only `devin airgap doctor` (air-gapped CLI builds) is documented — implies a separate air-gapped distribution exists, but no public install/usage docs. |
| U3 | "`system.json` full schema / supported keys beyond host+proxy." | ❓ Official page documents purpose and the two use cases; the complete key list is not published. |

## Source-doc citation quality

All claims in this harness were verified against docs.devin.ai first-party pages only (see README sources). Third-party coverage of Devin CLI/Desktop is sparse and frequently confuses three things: the **removed Cascade agent** vs **Devin Local**, the **editor MCP discovery file** vs the **agent MCP file**, and **Windsurf** vs **Devin** branding on identical settings. Any guide predating v3.9.19 or v3000.3 should be re-verified before relying on it.
