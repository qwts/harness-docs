# Vibe Agent Harness (Mistral Vibe Code)

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; admin config min CLI 2.24.2 / VS Code extension 1.16; docs site `docs.mistral.ai/vibe` main)

Vibe (docs root: “Vibe is Mistral's **unified agent** for productivity and coding tasks”) has two modes — **Work** (web/mobile productivity) and **Code** — and this set documents **Vibe Code**, the coding harness: “With read/write access to your filesystem, a shell, and a configurable set of tools, it can read files, run commands, write code, and open pull requests on your behalf, under your supervision.” Surfaces: “**One agent, three surfaces.** Vibe Code runs as the CLI, the VS Code extension, and Vibe Code Web,” plus ACP for other IDEs (JetBrains via AI Assistant; `vibe-acp`). Registry alias: roster `vibe` under `qwts-vibe-agent`. This reference does not document Vibe Work or the Mistral model APIs beyond the mode boundary.

Version anchors: shell-permission advisory fixes in **2.25.4** (2026-09-12, CVE-2026-87983…87988); admin config needs CLI **2.24.2** / extension **1.16**; release notes page (latest entries 2026-11-05). Credentials and account-specific secrets are excluded throughout.

**Verdict legend:** ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## In this folder

| File | Covers |
|---|---|
| [installation-auth-and-models.md](installation-auth-and-models.md) | Install (`mistral-vibe` via `uv`, installer script), `vibe`/`vibe-acp`, the three-way key precedence (shell → `~/.vibe/.env` → OS keyring), browser sign-in default, providers/`[[models]]` presets, offline OpenAI-compatible models |
| [configuration-and-admin-config.md](configuration-and-admin-config.md) | `config.toml` merge and trust-gated project layer, `VIBE_HOME` layout, key reference highlights, admin config (delivery, precedence, no-merge rule, RBAC, “distribution mechanism, not a security control”) |
| [agents-sessions-and-worktrees.md](agents-sessions-and-worktrees.md) | Built-in and custom agents (`agent_type`, subagents), interactive vs programmatic modes, session commands (`/resume`, `/branch`, `/rewind`, `/loop`, `/pr`), `--worktree` lifecycle |
| [safety-approvals-and-permissions.md](safety-approvals-and-permissions.md) | Layered controls, the four agents and smart approve (classifier, not a sandbox), trusted folders, per-tool permissions, bash allow/deny lists, outside-workdir policy, per-surface approval UX, advisory MAI-2026-003 |
| [mcp-skills-plugins-and-hooks.md](mcp-skills-plugins-and-hooks.md) | MCP transports and OAuth `vibe mcp add`, connectors vs MCP, Agent-Skills-spec skills (un-enforced `allowed-tools`), experimental Agent-Plugins-1.0 plugins with `ai.mistral.vibe/`, hooks (`pre_tool`/`post_tool`/`post_agent`), ACP clients |
| [web-sessions-and-automation.md](web-sessions-and-automation.md) | Vibe Code Web (projects, sessions, single-tenant sandbox, managed outbound network, GitHub attribution, prompt-injection model, Slack), automation boundaries, the Work-mode boundary |
| [corrections.md](corrections.md) | Claim-by-claim audit log (28 rows: 17 ❌ / 11 ⚠️) |

## Top corrections

1. **“Vibe Code asks before every file edit.”** ❌ Default agent `accept-edits` “Auto-approves file edits in the working directory.”
2. **“Vibe Code has a built-in plan agent/mode.”** ❌ “Vibe has no built-in `plan` agent” — missing custom plan agents fall back to `ask`.
3. **“Smart approve sandboxes the agent.”** ❌ “Smart approve is a classifier, not a sandbox.”
4. **“`auto-approve` is safe for daily use.”** ❌ “this agent can run arbitrary commands such as `rm -rf`”; admin `enable_auto_approve = false` hides it.
5. **“MCP tools use `mcp__server__tool`.”** ❌ “MCP tools are addressed as `{server_name}_{tool_name}`.”
6. **“Skill `allowed-tools` are enforced.”** ❌ “Vibe reads this field but doesn't enforce it.”
7. **“Admin config is a security boundary.”** ❌ “a **distribution mechanism, not a security control**”; org and workspace configs never merge.
8. **“Project `.vibe/` config always loads.”** ❌ Trust-gated: untrusted dirs “ignored … Vibe prints a warning.”
9. **“Vibe Code Web runs on your machine.”** ❌ Managed cloud sandbox that “cannot reach your local machine,” single-tenant, deleted at session end.
10. **“A Mistral API key is always required.”** ⚠️ “Browser-based sign-in is enabled by default” for Mistral-provider setups.
11. **“Mistral Code Enterprise JetBrains plugin is supported.”** ❌ “It's deprecated”; ACP is the supported path.
12. **“Claude Code plugin hooks run in Vibe.”** ❌ “Hooks in plugins written for Claude Code or Kimi Code don't run.”

## Quick reference

| Area | Fact |
|---|---|
| Product | Mistral Vibe = Work + Code modes; Code = CLI (`mistral-vibe`, `vibe` + `vibe-acp`) + VS Code extension + Vibe Code Web + ACP |
| Install | `curl -LsSf https://mistral.ai/vibe/install.sh \| bash` (uv) or `uv tool install mistral-vibe`; Windows via uv |
| Auth | Mistral API key: shell env → `~/.vibe/.env` → OS keyring; browser sign-in default for Mistral providers; `/status`, `/whoami` |
| Config | Two `config.toml` files (project `.vibe/config.toml` overrides user `~/.vibe/config.toml`); project trust-gated; `strict_config`, `vibe config schema`, `VIBE_HOME` |
| Admin config | Fetched to `~/.vibe/managed_config_cache.json`, precedence over all sources, org/workspace (no merge), `manage_code_config` RBAC |
| Agents | `ask`, `accept-edits` (default), `smart-approve`, `auto-approve`, `lean`; custom `.toml` in `~/.vibe/agents/` / `./.vibe/agents/`; `agent_type` agent/subagent; no built-in plan agent |
| Approvals | Per-tool `permission` always/ask/never; bash allow/deny prefix lists; outside-workdir ask/deny; `.env`/`.envrc` always ask unless allowlisted |
| Smart approve | Rules → session reuse → model classify → hold-back; classifier via `[utility_models].smart_approve`; “not a sandbox” |
| Sessions | `/resume`, `/branch`, `/rewind`, `/compact`, `/loop`, `/pr`, `/context`, `/reload`; `--resume`/`-c`; `&` → cloud session |
| Worktrees | `--worktree [name]` under `~/.vibe/worktrees/`; auto-clean when clean; macOS copy-on-write templates; `worktree_limit` 15 |
| Skills | Agent Skills spec `SKILL.md`; project `.vibe/skills`/`.agents/skills` (trusted) + user; `user-invocable`, `disable-model-invocation`; `allowed-tools` not enforced |
| Plugins | Experimental Agent Plugins 1.0 + `ai.mistral.vibe/`; built-in `vibe` plugin; pinned snapshots; Claude/Kimi plugin hooks unsupported |
| Hooks | `hooks.toml` (`pre_tool`/`post_tool`/`post_agent`); fnmatch/`re:` matchers; project wins; subagents inherit; exit-code + JSON contract |
| MCP | `vibe mcp add` (OAuth/`--api-key-env`/stdio), `[[mcp_servers]]`, transports http/streamable-http/stdio; tools named `{server}_{tool}`; admin `enable_mcp` |
| ACP | `vibe-acp`; JetBrains AI Assistant registry; Mistral Code Enterprise JetBrains plugin deprecated |
| Web | Cloud sandbox per session (single-tenant, deleted at end); managed outbound-only network; GitHub App attribution; Slack entry point |
| Models | `[[providers]]`/`[[models]]` presets; offline via OpenAI-compatible API (Devstral Small 2, Mistral Small 4; vLLM) |

## Official sources

- [Vibe overview](https://docs.mistral.ai/vibe) · [Vibe Code overview](https://docs.mistral.ai/vibe/code/overview) · [CLI docs](https://docs.mistral.ai/vibe/code/cli/install-setup) · [Safety and permissions](https://docs.mistral.ai/vibe/code/safety-approvals-permissions) · [Vibe Code Web](https://docs.mistral.ai/vibe/code/vibe-code-web/get-started) · [Configuration reference](https://docs.mistral.ai/vibe/code/cli/configuration-reference) · [Security advisory MAI-2026-003](https://docs.mistral.ai/resources/security-advisories/MAI-2026-003) · [Release notes](https://docs.mistral.ai/resources/release-notes)
