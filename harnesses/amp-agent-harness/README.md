# Amp agent harness

**Verified:** 2026-11-15 against the official Amp documentation at ampcode.com/docs  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** CLI (interactive and execute mode), user/workspace/managed settings, model routing, AGENTS.md, skills, plugins, MCP, tools and permissions, subagents, threads, runners, Orbs, Puck, enterprise controls, and the SDK  
**Provenance:** Official Amp sources unless explicitly marked otherwise

Amp is a coding agent and development environment: one agent and one set of threads served across the web app, desktop/mobile apps, and a terminal CLI, with execution decoupled from viewing — a thread can run locally, on a runner (a machine you already have), or in an Orb (a cloud sandbox Amp creates). Amp is deliberately opinionated (“No backward compatibility, no legacy features”), and its permission model is inverted relative to most harnesses: it does not ask for approval before running tools by default; control comes from plugins and MCP/tool policies, not an approval prompt.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, settings, and model routing](configuration-settings-and-model-routing.md) | Settings files and precedence, managed settings, key catalog, accounts and updates, connections and routing order |
| [Instructions, skills, and plugins](instructions-skills-and-plugins.md) | AGENTS.md discovery, @-mentions, skill locations and masking, plugin API and lifecycle, plugin agents |
| [MCP, tools, and permissions](mcp-tools-and-permissions.md) | MCP server config, trust and per-tool policies, tool disabling, the no-approval default, enterprise MCP registry |
| [Subagents and custom agents](subagents-and-custom-agents.md) | Task and Read Thread subagents, isolation, oracle/librarian/finder plugins, custom agents and modes |
| [Surfaces: CLI, Orbs, threads, and SDK](surfaces-cli-orbs-threads-and-sdk.md) | Interactive/execute mode, runners and cross-client access, Orbs, thread lifecycle, Puck, apps, SDK |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Amp asks for approval before running terminal commands.”** By default it does not ask before running any tool; control comes from plugins (`tool.call` policies), MCP permission rules, and enterprise registry enforcement.
- ❌ **“Amp is a VS Code extension.”** Amp is a web app, native apps for macOS/Windows/Linux/iOS/Android, and a CLI; IDEs connect through `ide connect` rather than an Amp extension.
- ❌ **“Threads live on your machine.”** Every thread has an `ampcode.com/threads/T-…` URL and opens across all clients; execution location is separate from where you view it.
- ❌ **“The Oracle is a built-in tool that edits files.”** The oracle is an optional plugin (`@amp/oracle`) that reviews and advises and never edits files.
- ❌ **“Workspace settings are overridden by user settings.”** Workspace settings override user settings; only keymap entries are reversed.
- ❌ **“Amp supports local models over localhost.”** Amp sends model requests from its own servers, so providers must allow Amp’s egress IPs; localhost endpoints are not a documented connection type.
- ❌ **“MCP servers in `.amp/settings.json` run automatically when you open a project.”** Workspace-configured servers require explicit approval before they run.
- ❌ **“MCP servers created by Puck can execute its skills’ scripts.”** Puck has no orb: it reads skill instructions and files but cannot run scripts, and plugins do not load in Puck.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| User settings | `~/.config/amp/settings.json` (or `.jsonc`); Windows `%USERPROFILE%\.config\amp\settings.json` |
| Workspace settings | Nearest `.amp/settings.json` / `.jsonc`, searched upward from cwd to repo root |
| Managed settings (workspace policy) | One JSON document admins add in Workspace Settings → Managed Settings (cloud-hosted, revision-tracked) |
| Managed settings (machine-level) | `/Library/Application Support/ampcode/managed-settings.json` (macOS), `/etc/ampcode/managed-settings.json` (Linux), `%ProgramData%\ampcode\managed-settings.json` (Windows) |
| Settings precedence | Workspace settings (`.amp/`) override user settings; keymap entries reverse that. The admin policy document beats local settings on scalars, merges lists workspace-first and objects key-by-key; the machine-level file merges the same way and takes precedence over the policy document |
| Key namespace | All settings use the `amp.` prefix |
| Keys blocked in the policy document | `amp.url`, `amp.proxy`, `amp.mcpServers`, `amp.workspaces`, `amp.mcpTrustedServers` — this restriction does not apply to repo `.amp/settings.json`, which can set `amp.mcpServers` |
| MCP config key | `amp.mcpServers` (stdio: `command`/`args`/`env`; remote: `url`/`headers`) |
| MCP loading order | CLI `--mcp-config` → workspace → user → skills |
| Default tool approval | None — Amp runs tools without asking |
| Guidance files | `AGENTS.md` (cwd, parents to `$HOME`, subtrees on read, system-wide); `CLAUDE.md`/`AGENT.md` as fallback |
| Skill locations | `.agents/skills/` in project; `~/.config/agents/skills/`, `~/.agents/skills/`, `~/.config/amp/skills/`, `.claude/skills/`, `~/.claude/skills/`, `~/.claude/plugins/cache/`, `amp.skills.path`, personal/workspace/official repos — first name wins |
| Plugin locations | `.amp/plugins/` (project), `$XDG_CONFIG_HOME/amp/plugins/` or `~/.config/amp/plugins/` (system), personal and workspace repos — collision order project → system → personal → workspace |
| Git attribution | `Amp-Thread-ID:` trailer and `Co-authored-by: Amp <amp@ampcode.com>`, both enabled by default |
| Headless run | `amp -x "prompt"` (execute mode); `amp -ox` runs the prompt in an orb |
| CI auth | `AMP_API_KEY` (`sgamp_…` token); takes precedence over saved accounts until unset |
| Runner | `amp --no-tui --runner-id <id>` |
| Thread export | Append `.md` to the thread URL; `amp threads markdown` / `amp threads export` |

## Official sources

- [Docs index](https://ampcode.com/docs)
- [CLI](https://ampcode.com/docs/cli) · [CLI settings](https://ampcode.com/docs/cli/settings) · [Execute mode](https://ampcode.com/docs/cli/execute-mode) · [Runners](https://ampcode.com/docs/cli/runners) · [Cross-client access](https://ampcode.com/docs/cli/remote-control)
- [Tools](https://ampcode.com/docs/tools) · [Models and subagents](https://ampcode.com/docs/models-and-subagents) · [Model routing](https://ampcode.com/docs/customize/model-routing)
- [AGENTS.md](https://ampcode.com/docs/customize/agents-md) · [Skills](https://ampcode.com/docs/customize/skills) · [Plugins](https://ampcode.com/docs/customize/plugins) · [MCP](https://ampcode.com/docs/customize/mcp)
- [Threads](https://ampcode.com/docs/threads) · [Orbs](https://ampcode.com/docs/orbs) · [Puck](https://ampcode.com/docs/puck)
- [Enterprise: allowed models](https://ampcode.com/docs/enterprise/allowed-models) · [MCP registry allowlist](https://ampcode.com/docs/enterprise/mcp-registry-allowlist) · [Minimal data retention](https://ampcode.com/docs/enterprise/minimal-data-retention)
- [SDK](https://ampcode.com/docs/sdk)
