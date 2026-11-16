# Roo Code Agent Harness

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site `roocodeinc.github.io/Roo-Code` main)

> ⚠️ **Sunset product.** “All Roo Code products will be shut down on May 15, 2026. This includes the Roo Code VS Code Extension, Roo Code Cloud, and Roo Code Router.” That date has passed: Cloud and Router are shut down, unused balances were refunded, and the extension repository was archived. This reference documents the final shipped state (v3.54.0, marketplace `RooVeterinaryInc.roo-cline`) for audit and migration purposes; the org roster still lists `roocode` as an active harness, but the vendor product is dead. Successors named by the vendor: Cline, roomote.dev, and the community fork ZooCode.

Roo Code (repo `RooCodeInc/Roo-Code`, Apache-2.0) was “an AI-coding suite of products” — a model-agnostic VS Code extension (“Your AI-Powered Dev Team, Right in Your Editor”) plus the Roo Code Cloud web platform with autonomous Cloud Agents, the Roo Code Router provider, and an official but never-released-from-prerelease CLI (`roo`, v0.1.x, macOS/Linux). It originated as a Cline fork.

**Verdict legend:** ✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## In this folder

| File | Covers |
|---|---|
| [installation-and-providers.md](installation-and-providers.md) | Install surfaces (Marketplace/Open VSX/VSIX + the `roo` CLI prerelease), provider landscape and the model-agnostic stance, Router, API configuration profiles, settings export/import, local models, rate limits and cost estimates |
| [modes-and-workflows.md](modes-and-workflows.md) | The five built-in modes and tool groups, custom modes, sticky models, Boomerang/Orchestrator orchestration, worktrees, message queueing, context mentions |
| [instructions-skills-and-commands.md](instructions-skills-and-commands.md) | Rules directories and `.roorules` fallbacks, skills with progressive disclosure, slash commands, Marketplace, prompt structure |
| [tools-approvals-and-safety.md](tools-approvals-and-safety.md) | Tool groups and always-available tools, auto-approval tiles and the queued-message implicit-approval caveat, checkpoints, `.rooignore`, shell integration, context condensing, codebase indexing, experimental features |
| [mcp.md](mcp.md) | Global `mcp_settings.json` vs project `.roo/mcp.json` precedence, transports, `alwaysAllow`/`disabled`, Marketplace MCPs |
| [cloud-and-sunset.md](cloud-and-sunset.md) | Roo Code Cloud, the Cloud Agent team (Explainer/Planner/Coder/Reviewer/Fixer), GitHub/Slack integrations, preview environments, Team plan, credits, the Router, and the sunset timeline |
| [corrections.md](corrections.md) | Claim-by-claim audit log (29 rows: 19 ❌ / 10 ⚠️) |

## Top corrections

1. **“Roo Code is actively developed.”** ❌ Shut down May 15, 2026 — Extension, Cloud, and Router.
2. **“Roo Code is a CLI.”** ⚠️ The documented surface was “a VS Code extension” (plus Cloud), but an official prerelease CLI (v0.1.17, `roo "What is this project?"`) also shipped before the sunset.
3. **“Queued messages respect approval settings.”** ❌ “Roo implicitly approves whatever would normally require your confirmation …—even if auto-approval is disabled.”
4. **“Custom tools ask before running.”** ❌ “Custom tools are **automatically approved** when enabled.”
5. **“Settings exports are safe to share.”** ❌ “this includes **API keys in plaintext**.”
6. **“Checkpoints are off by default.”** ❌ “**Checkpoints are enabled by default**” (contrast with several peer harnesses).
7. **“MCP config is project-only.”** ❌ Global `mcp_settings.json` plus project `.roo/mcp.json`; “the **project-level configuration takes precedence**.”
8. **“Orchestrator needs a custom Boomerang mode.”** ❌ “now a built-in mode.”
9. **“Architect can edit any file.”** ❌ “restricted `edit` (markdown files only).”
10. **“Roo Code Cloud is all paid.”** ⚠️ Sync/sharing/analytics were free; Cloud Agents were credit-based and “available in all Roo Code Cloud plans, including the free one.”
11. **“Cloud Agents run locally.”** ❌ “they work on isolated branches and deliver work via PRs.”
12. **“Context condensing is opt-in.”** ❌ “enabled by default.”

## Quick reference

| Area | Fact |
|---|---|
| Product | VS Code extension (`RooVeterinaryInc.roo-cline`, Apache-2.0) + Roo Code Cloud + Router — all sunset 2026-05-15; prerelease `roo` CLI (v0.1.17) also existed |
| Modes | `💻 Code`, `❓ Ask` (read+mcp only), `🏗️ Architect` (markdown-only edit), `🪲 Debug`, `🪃 Orchestrator` (delegates via `new_task`); sticky models; mode persists across sessions |
| Custom modes | Global or project YAML (slug/name/description/roleDefinition/groups/whenToUse/customInstructions); import/export; Marketplace distribution |
| Approvals | Master Enabled toggle + per-permission tiles (`Cmd/Ctrl+Alt+A`, `roo-cline.toggleAutoApprove`); command execution rated High risk |
| Queue caveat | Processing a queued message implicitly approves confirmations, even with auto-approval off |
| Checkpoints | On by default; shadow Git independent of project config; captured at task start and before file edits (not commands) |
| Rules | `~/.roo/rules/`, `.roo/rules/`, `.roo/rules-{modeSlug}/`; `.roorules` files as fallbacks; workspace beats global |
| Skills | `.roo/skills/` + `~/.roo/skills/`; progressive disclosure via SKILL.md frontmatter; mode-targetable |
| Slash commands | `.roo/commands/` / `~/.roo/commands/`; `run_slash_command` tool experimental |
| MCP | `mcp_settings.json` (global) + `.roo/mcp.json` (project wins); STDIO/Streamable HTTP/SSE; `alwaysAllow`, `disabled`; Marketplace |
| Indexing | Tree-sitter + embeddings + Qdrant → `codebase_search` |
| `.rooignore` | gitignore syntax; hot-reloaded; self-protecting |
| Cloud | Task sync/share/analytics free; Cloud Agents (Explainer/Planner/Coder/PR Reviewer/PR Fixer) credit-based with BYO keys; isolated branches + PRs; Team plan with config enforcement |
| Sunset | 2026-05-15: Cloud+Router shut down, balances refunded, repo archived; → Cline / roomote.dev / ZooCode |

## Official sources

- [Docs site](https://roocodeinc.github.io/Roo-Code/) · [GitHub repo (archived)](https://github.com/RooCodeInc/Roo-Code) · [Docs repo](https://github.com/RooCodeInc/Roo-Code-Docs) · [Sunset notice](https://roocodeinc.github.io/Roo-Code/sunset) · [CLI v0.1.17 release notes](https://github.com/RooCodeInc/Roo-Code/releases/tag/cli-v0.1.17) · [README disclaimer](https://github.com/RooCodeInc/Roo-Code#disclaimer)
