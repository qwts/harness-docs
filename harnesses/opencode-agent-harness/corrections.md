# Corrections — common misinformation about OpenCode

No upstream draft existed for this harness; this file audits claims commonly repeated about OpenCode, each checked 2026-10-09 against opencode.ai/docs.

## FALSE

| ID | Claim | Reality |
|---|---|---|
| F1 | "Config files replace each other — project config shadows global entirely." | Configs **merge**; later sources only override on conflicting keys. Precedence has 8 tiers (remote → global → OPENCODE_CONFIG → project → .opencode dirs → OPENCODE_CONFIG_CONTENT → managed → MDM). |
| F2 | "Put config in `.opencode/config.json`." | Project config is `opencode.json` at repo root. `.opencode/` is a directory of subdirs (`agents/`, `commands/`, `plugins/`, `skills/`). |
| F3 | "opencode is a fork/rebrand of Claude Code or Windsurf." | Independent SST/anomalyco project (MIT). `.claude` files are only optional compat fallbacks, disable-able via `OPENCODE_DISABLE_CLAUDE_CODE*`. |
| F4 | "MCP OAuth needs a client ID/secret configured up front." | Auto-detects 401 → OAuth flow with Dynamic Client Registration (RFC 7591); preregistration is optional. `oauth: false` disables it. |
| F5 | "Deny rules are permanently safe once written." | Granular rules resolve by **last matching pattern wins regardless of effect** — a later `allow` overrides an earlier `deny`. `--auto` only preserves a deny when it is the final resolution, so place deny rules last. |

## PARTIAL

| ID | Claim | Reality |
|---|---|---|
| P1 | "`tools` config controls tools." | Deprecated since v1.1.1 → merged into `permission` (allow/ask/deny + granular glob rules). Still parses for back-compat: `true` ≈ `{"*":"allow"}`. |
| P2 | "Use singular `.opencode/agent/` dirs." | Plural (`agents/`, `commands/`, `plugins/`, `skills/`, `modes/`, `tools/`, `themes/`) is canonical; singular still works as a back-compat alias. |
| P3 | "All AGENTS.md/CLAUDE.md files load together." | First match wins per category: project `AGENTS.md` shadows `CLAUDE.md`; `~/.config/opencode/AGENTS.md` shadows `~/.claude/CLAUDE.md`. Extra files go in `instructions`. |
| P4 | "Subagents run on the global `model`." | Unset → subagents inherit the invoking agent's model; only primary agents use the global `model`. |
| P5 | "`~` in a permission pattern makes a path local." | Home expansion only rewrites the pattern — external paths still need `external_directory` allowance. |
| P6 | "opencode is terminal-only." | TUI is primary, but there are `opencode run`, `opencode serve`, a desktop app (beta), a web app, and an IDE extension. |
| P7 | "Custom commands need JSON config." | A markdown file under `commands/` with frontmatter suffices; `command.*` JSON is the alternative. |
| P8 | "`--auto` runs everything." | Auto-approves only non-denied actions; explicit `"deny"` still enforced. |
| P9 | "npm plugins need manual install." | `plugin: [...]` names are auto-installed by Bun at startup to `~/.cache/opencode/node_modules/`. |
| P10 | "Project config is found only in CWD." | Startup walks up to the nearest git directory to find `opencode.json`. |
| P11 | "Remote org config is authoritative." | It's the *lowest* tier — everything local can override it; only managed/MDM layers are truly binding. |

## UNVERIFIED

| ID | Claim |
|---|---|
| U1 | Desktop app uses identical config surface to TUI (beta, docs don't specify a separate config file). |
| U2 | `opencode web` / IDE extension feature parity with the TUI. |
| U3 | Per-provider `options` beyond `baseURL` (docs only formally document `baseURL`/`blacklist`/`whitelist`; provider-specific options exist in schema but are per-provider). |
