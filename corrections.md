# Corrections Log — Audit of the Source Draft

Claim-by-claim audit of "Architecture and Configuration Schema of the Claude Agent Harness" (Gemini-produced draft). Verified 2026-10-09 against official Claude Code docs (docs.claude.com / code.claude.com), MCP official docs (modelcontextprotocol.io), and independent sources. Only entries with a non-✅ verdict are listed here; everything else in the draft checked out and is organized into the sibling topic files.

## ❌ False claims (do not rely on)

| # | Draft claim | Reality |
|---|---|---|
| F1 | "If the User Scope and the Project Scope define the same server alias, the User Scope definition … overrides the project-level definition." | **Reversed.** MCP scope precedence is **local > project > user** (> plugin-provided > claude.ai connectors). Project beats user. The winning entry is used whole; fields are not merged; conflicts are flagged in `claude mcp list` / `/mcp`. |
| F2 | Desktop-imported servers are "written directly to the global user configuration file, making the imported servers available to all CLI sessions." | `claude mcp add-from-claude-desktop` writes to **local scope by default** (per-project entry in `~/.claude.json`). `--scope user` is an explicit opt-in. |
| F3 | "Every server definition within a CLI configuration file must include a specific transport declaration key … the CLI will reject the configuration [without it]." | Omitted `type` is read as **stdio**. `type` is required only for remote transports; `url` without `type` is the actual error case (fix: add `"type": "http"`/`"sse"`/`"ws"`). |
| F4 | Claude Code memory uses "specific XML citation tags when referencing this memory, generating a decay signal that ensures stale claims are deprecated over time"; agents must "trigger these citation mechanics" when migrating config. | No XML citation or decay mechanism exists in any official source. Memory = plain Markdown + YAML frontmatter (`name`, `description`, `type`, `modified`) in `~/.claude/projects/<project>/memory/`. Config migration has no interaction with the memory system. Appears **fabricated**. |
| F5 | (Implicit) The ecosystem "actively rejects recursive searches through arbitrary directories for configuration data" across both surfaces. | True for config files (fixed paths only), but the instruction layer **does** recurse: CLAUDE.md files load up the directory tree, and nested rules load on demand. Overgeneralized. |

## ⚠️ Partially true / overstated

| # | Draft claim | Correction |
|---|---|---|
| P1 | Linux Desktop path `~/.config/Claude/claude_desktop_config.json` presented as an official target. | Claude Desktop officially ships for **macOS and Windows only**. The Linux path applies to unofficial/Electron-wrapped builds. On Linux, Claude Code is the supported MCP surface. |
| P2 | Desktop `command` "must be absolute … to prevent execution failures." | Absolute paths are a strong **recommendation** (GUI-session PATH differs from shell PATH), not schema-enforced. |
| P3 | `allowAllClaudeAiMcps` described as overriding "all user-defined and local MCP connectors." | It allows **claude.ai cloud connectors** to load alongside a deployed `managed-mcp.json` — it does not unsuppress user-defined/local servers. (The setting itself is real; the draft's semantics are wrong.) |
| P4 | MCP operates "strictly on a request-response basis"; no autonomous push. | Outdated: servers push `list_changed` notifications (no restart), and opt-in **channels** (`claude/channel` capability, `--channels` flag) let servers push session messages. |
| P5 | "The harness requires explicitly defined terminal states" / will "continuously attempt to find supplementary actions." | Good prompt-engineering guidance, but it is not a harness-enforced mechanism. Reframed as guidance. |
| P6 | Managed scope "suppresses all user-defined and local MCP connectors unless explicitly overridden by `allowAllClaudeAiMcps`." | `managed-mcp.json` exclusivity is real (add commands fail; `--mcp-config` exits at startup), but the exception clause mischaracterizes `allowAllClaudeAiMcps` (see P3). |
| P7 | "Import … automatically injects the standard input/output transport declaration into the server object." | Plausible effect, unverifiable implementation detail; functionally moot since typeless entries parse as stdio. The elaborate "deterministic translation algorithm" narrative is illustration, not documented spec. |
| P8 | Import is "natively supported on macOS and Windows Subsystem for Linux." | ✅ as stated, but note the draft's framing hides that native **Windows** (non-WSL) is excluded, where the `npx.cmd`/`cmd /c` workaround applies. |

## 💬 Opinion presented as architecture (kept, relabeled)

| # | Draft section | Assessment |
|---|---|---|
| O1 | DAG orchestration, parallel worktrees, cascading abort trees, "linear loop = critical failure pattern" | Sound engineering guidance for agent orchestration, sourced from blog commentary. Not documented harness behavior. |
| O2 | Layered system-prompt conflict behavior ("erratic and unpredictable") | Official docs do warn Claude may pick arbitrarily between contradictory instructions; "erratic" is editorial. |
| O3 | Comparative performance/economics (fewer actions, exponentially more tokens, cache-discount economics, "optimized for native ecosystem") | Single-blog opinion, not independently verifiable. Design facts beneath it (heavy per-step context, tool search) are real; benchmarks are not. |
| O4 | CI/CD connector vignette (pipeline telemetry, deployment-stage timing analysis) | Illustrative third-party integration example; the approval-gating principle behind it is genuine. |

## ✅ Notable claims that survived verification (surprising ones)

- `"type": "sdk"` is a real reserved transport, rejected ("skipped") outside SDK host applications — sounded like a hallucination but is confirmed in official docs.
- `streamable-http` as a JSON alias for `http` — confirmed.
- Auto-memory type taxonomy **user / feedback / project / reference** — confirmed (frontmatter `type` field).
- Managed > CLI flags > local > project > user settings precedence (draft Table 6) — confirmed, matching official docs.
- Desktop full-quit requirement, Windows tray exit, `npx.cmd`/`.cmd` variant requirement — confirmed.
- Tool search threshold switching and on-demand schema retrieval — confirmed (default on; `ENABLE_TOOL_SEARCH=auto` = 10% context budget by default).
- PreToolUse hooks as deterministic blocking barriers — confirmed.
- Project-scope `.mcp.json` pending-approval security gating — confirmed (plus a workspace-trust layer the draft missed: a cloned repo can't approve its own servers).

## Source-doc citation quality

The draft's own footnotes are weak: the Reddit post is cited twice, two MCP-doc citations are duplicated, and the memory/DAG/economics claims trace to a single personal blog ("Model-Harness-Fit") that does not document harness internals. Several strong claims (F1–F4) contradict the official documentation the draft elsewhere cites. Conclusion: **structural/CLI material is largely trustworthy; behavioral/memory/economic claims required correction.**
