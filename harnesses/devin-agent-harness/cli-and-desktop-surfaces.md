# Surfaces: Devin CLI & Devin Desktop (Verified)

> Verdicts inline: ✅ TRUE / ⚠️ PARTIAL / ❌ FALSE / 💬 OPINION / ❓ UNVERIFIED

Verified 2026-10-09 against `cli/*` and `desktop/*` docs (docs.devin.ai), including the Desktop changelog through v3.9.19.

## Devin CLI — install & auth

| Channel | Command |
|---|---|
| macOS / Linux / WSL | `curl -fsSL https://cli.devin.ai/install.sh \| bash` |
| Homebrew (macOS) | `brew install --cask devin-cli` (upgrade: `brew upgrade --cask devin-cli`) |
| Windows | `irm https://static.devin.ai/cli/setup.ps1 \| iex` in **PowerShell** (not Git Bash/CMD); or x86_64/ARM64 installer EXEs |
| Devin Desktop bundle | Command Palette → **Install Devin CLI** (admin must enable "Install Devin CLI in Devin Desktop" in team settings; Enterprise plans) |

Auth: `devin auth login` (browser; `--force-manual-token-flow` for SSH), `devin auth logout`, `devin auth status`. Enterprise: access gated by RBAC role permission **Use Devin CLI**; `system.json` can pin host/account.

## CLI command surface

| Command | Purpose |
|---|---|
| `devin` | Interactive TUI in current project |
| `devin -r` / `/resume`, `devin ls` | Resume/list sessions |
| `devin rm <id-or-name>` | Delete a session (refuses while open elsewhere) |
| `devin doctor` | Validate custom subagent frontmatter |
| `devin desktop`, `devin .` | Open Devin Desktop (on path) |
| `devin acp` | ACP server over stdio — see below |
| `devin mcp`, `devin plugins`, `devin rules` | See [mcp-and-extensibility.md](mcp-and-extensibility.md) |
| `devin ssh`, `devin cloud drs` | Cloud bridge — see below |
| `devin airgap doctor` | Air-gapped builds only: license, config paths, model endpoint probes; `--json` |

Key flags: `--model <fuzzy-name>`, `--permission-mode <normal|accept-edits|smart|bypass>`, `--sandbox`, `--cloud`, `-p` (print/non-interactive), `--prompt-file`.

Useful slash commands: `/config`, `/model`, `/mode <name>` (+ `/normal` `/accept-edits` `/smart` `/plan` `/bypass`/`/yolo`/`/dangerous`, `/ask`), `/compact`, `/context`, `/usage`, `/session-stats` (`/stats`), `/btw` (parallel side-chat), `/recap`, `/rename`, `/bug`, `/update`, `/shortcuts` (rebindable keys), `/feedback`, `/org`. Instant commands run while the agent works instead of queueing.

## Cloud bridge (local CLI → Devin Cloud)

- `devin --cloud -p "prompt"` — one-shot cloud session, prints response, exits; resume with `devin --cloud -r`. Also `--prompt-file`.
- Cloud slash commands: `/cloud`, `/repo`, `/platform`, `/model`, `/open [web|desktop]`, `/ssh`, `/pickup` `/handoff` (check out PR branch + continue locally), `/rename`, `/archive`.
- `devin ssh <session-id-or-url>` — shell on the cloud VM (wraps system ssh; `-L` passthrough). Plain `ssh devin-<id>@ssh.devin.ai` / `scp` also work via a browser-approved gateway.
- `devin cloud drs` — Declarative Repo Setup: `blueprint-*`, `sandbox-create --repo`, `run --devin-id --command`, `build[-start|-wait|-logs]` (NDJSON), `secret-create`.

## ACP (Agent Client Protocol)

✅ TRUE — `devin acp` runs the harness as a **JSON-RPC server over stdio** for ACP-aware editors (Zed, Devin Desktop's Devin Local). Not interactive; credentials from `WINDSURF_API_KEY`, else `devin auth login` store, else ACP `authenticate`. Flags: `--agent-type`, `--model` / `DEVIN_MODEL` (fuzzy names accepted). This is the embedding contract for third-party editor hosts — analogous in role to Codex's app-server.

## Models

- Selection: `--model`, `/model`, `agent.model` in user config; fuzzy names (slug/alias/partial).
- Routers: **Fusion** (frontier lead + cost-efficient sidekick pairing) and **Adaptive** (per-task router, dynamic pricing) — admin-gated.
- Team controls: model allowlist, default model (until user picks their own — allowlist wins over pinned default), default subagent model.

## Devin Desktop

✅ TRUE — Devin Desktop is the **rebranded Windsurf editor** ("Windsurf" and "Devin Desktop" are the same product; legacy Windsurf Enterprise and Devin Enterprise write the same team-settings store).

- **Devin Local** is the primary local agent and **shares the Devin CLI harness** — same skills format/discovery, rules engine, permissions. It launches the CLI's ACP server (`devin acp`) internally.
- ❌ STALE INFO RISK — **Cascade was removed in v3.9.19 (2026-09-08)**; Devin Local is the only bundled agent. Any doc describing Cascade as current is outdated (its memories/workflows aren't supported by Devin Local; migrate via the wizard → skills).
- Agent locations: **Local**, **Worktree**, **Cloud** (top-level picker); sessions appear in the **Agent Command Center** Kanban; **Spaces** group sessions/PRs/files per task; OS notifications via `devin.agentNotifications` (off by default).
- Restricted Mode workspaces disable Cascade/Devin Local/every ACP agent.
- Desktop MCP files: agent (Cortex-managed) `~/.config/devin/mcp_config.json` (`$XDG_CONFIG_HOME/devin/` honored) / `%AppData%\devin\mcp_config.json` — **separate** from editor discovery file `~/.codeium/windsurf/mcp_config.json` (`windsurf` source under `chat.mcp.discovery.enabled`).
- Admin surfaces (three): **team settings** (account-following, server-side), **device policies** (MDM: Group Policy, macOS profiles, `/etc/vscode/policy.json`), **system-level files** (e.g. system hooks.json). All non-overridable by users.
