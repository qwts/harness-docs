# Security, permissions, and sessions

**Verified:** 2026-11-15

## Approvals and YOLO

Dangerous-command approval modes: `manual` (prompt a person), `smart` (an auxiliary LLM classifies and decides — `decided_by: "aux_llm"`), `off`. Prompts time out after `approvals.timeout` (default 300 s); on messaging platforms the prompt becomes buttons plus `/approve [session|always]` and `/deny`. Destructive slash commands (`/new`, `/clear`, `/undo`, `/exit --delete`) open a three-choice modal unless `approvals.destructive_slash_confirm: false` or an inline `now`/`--yes`/`-y` skips it.

YOLO (`--yolo`, `/yolo`, `HERMES_YOLO_MODE`) skips dangerous-command approval prompts, but it is not a master override: a hardline blocklist stays as a floor, and `approvals.deny` user globs “are evaluated before YOLO”. Sudo uses a masked password prompt on interactive parents (cached for the session; `SUDO_PASSWORD` for messaging/headless); “delegated subagents cannot open a password prompt.”

## File write safety

File writes pass protected-path checks and an optional `HERMES_WRITE_SAFE_ROOT` confinement root; blocked hand edits to `jobs.json` are the documented example of why the “file-mutation verifier footer is the authoritative signal that nothing was saved.”

## Gateway authorization

The messaging gateway checks inbound senders in this order: platform-allow-all → DM-pairing → platform-allowlist → global-allowlist → global-allow-all → default-deny. Slash commands split admin/user per platform: admins get everything, regular users only `user_allowed_commands` (floor: `/help`, `/whoami`); “if `allow_admin_from` is unset for a scope, that scope stays in unrestricted backward-compat mode.”

## Injection scanning

- **Context files** (`.hermes.md`, `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, and progressively discovered subdirectory copies) are scanned for prompt-injection, exfiltration, and invisible-Unicode patterns; a hit blocks the file: “[BLOCKED: AGENTS.md contained potential prompt injection …]”. Your own `SOUL.md` in `HERMES_HOME` is the exception — a hit logs a warning and loads anyway (a profile-distribution-owned `SOUL.md` is still blocked).
- **Memory entries** are scanned before acceptance (injection, exfiltration, invisible Unicode).
- **Cron prompts** are scanned at create/update time.
- **MCP tool results** have invisible Unicode TAG characters (U+E0000–U+E007F) stripped — “a classic prompt-injection smuggling channel” — and protocol-reserved `_meta` keys dropped.

## Isolation and egress

Container backends harden by default: read-only root filesystem (Docker), `--security-opt no-new-privileges` (“Block privilege escalation”), `--pids-limit 256`, size-limited `/tmp` and no-exec `/var/tmp` tmpfs mounts, and `--cap-drop ALL` — with three capabilities added straight back in the same base flags: `DAC_OVERRIDE` (“Root can write to bind-mounted dirs”), `CHOWN`, and `FOWNER` (“Package managers need file ownership”); `SETUID`/`SETGID` join “conditionally when the container starts as root and an init/entrypoint must drop privileges (the s6 privilege-drop path)” and are skipped for containers already running as a non-root `--user`. Persistent mode bind-mounts `/workspace` and `/root` from the profile’s sandbox directory. The egress proxy (`/egress`) fronts provider traffic from Docker. MCP stdio servers get only explicitly configured `env` plus a safe baseline — Hermes “does not blindly pass your full shell environment.” MCP OAuth tokens cache at `~/.hermes/mcp-tokens/<server>.json` with 0o600 perms; refresh tokens are dropped rather than sent to a changed issuer, and RFC 9207 `iss` mismatches are rejected (one documented Figma exemption).

## Sessions

Sessions persist in **SQLite** (`~/.hermes/state.db`, WAL journal, FTS5 index) — metadata, message history, compression lineage, and search indexes. Docs list 25 session sources (CLI, TUI, desktop, gateway platforms, ACP, cron, kanban workers, API, …).

- **Resume:** `hermes --resume <id|title>` (latest `#N` continuation of a titled lineage), `hermes -c` (most recent), `--resume latest --in ./dir`, `-c <title> --create-if-missing`.
- **Continuity:** messaging chats are “deliberately one continuous session that survives restarts, gateway crashes, and machine reboots”; auto-titles name sessions, and compression rotates lineages (`old_session_id` → new, surfaced as `#2`, `#3`).
- **Compression:** automatic at 50% of the context window; “middle turns are summarized while the first 3 and last 20 turns are always preserved.” `/compress here [N]` keeps the last N exchanges verbatim; `/compress focus <topic>` narrows a full summary.
- **Exports:** jsonl, md, qmd, html, and HF-trace formats; `session_search` runs raw FTS5 — “no LLM summarization, no truncation”.
- **Pruning:** off by default; `retention_days: 90` applies only when enabled. There are no idle or daily resets.

## Checkpoints and rollback

“Checkpoints are opt-in as of v2 — most users never use /rollback.” Enabled per-session (`hermes chat --checkpoints`) or via `checkpoints: enabled: true`, they snapshot into a shared shadow git repo under `~/.hermes/checkpoints/store/` — “the real project `.git` is never touched.”

- Captured before `write_file`/`patch` and destructive terminal commands (“rm, rmdir, cp, install, mv, sed -i, truncate, dd, shred, output redirects (>), and git reset/clean/checkout”), at most one per directory per turn.
- `/rollback <N>` restores only files Hermes changed (a content-hash ledger tracks agent writes — “↷ Kept your hand-edits: …”); `--all` overwrites hand-edits; `diff <N>` previews; a single file can be restored. Restores take a pre-rollback snapshot (“undo the undo”).
- Limits: nested git repos are captured as gitlinks only; container backends get no checkpoints; >50,000-file directories, `/`, and `$HOME` are skipped; 10 MB per file, 500 MB per store; auto-prune never deletes orphan entries (explicit `hermes checkpoints prune` only).

## Git worktrees

`hermes -w` (and `/worktree new`) run sessions in disposable worktrees under `<repo>/.worktrees/`, branched from the freshly-fetched remote tip. Trees are kept when they hold uncommitted changes, unpushed commits, an active loop/heartbeat/goal, an enabled cron job, or another attached viewer; a sweep runs at startup and every 6 hours, archiving before removing (`refs/hermes/archive/<tree>`, restorable with `hermes worktree restore`). Safety guarantees: “Uncommitted changes and unique unpushed commits are never deleted without a restorable archive”; patch-equivalent merged commits count as merged; trees in use are re-checked immediately before removal; untracked-only scratch is archived to `~/.hermes/archive/worktree-prune/`; branches are content-gated, “not name-gated.”

## Official sources

- [Security](https://hermes-agent.nousresearch.com/docs/user-guide/security) · [Sessions](https://hermes-agent.nousresearch.com/docs/user-guide/sessions) · [Checkpoints & Rollback](https://hermes-agent.nousresearch.com/docs/user-guide/checkpoints-and-rollback) · [Git Worktrees](https://hermes-agent.nousresearch.com/docs/user-guide/git-worktrees) · [Which File Does What?](https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what)
