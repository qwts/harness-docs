# Surfaces: IDE, CLI, Web, Mobile (+ Crew orchestrator)

## One harness, four front ends

- **✅** Kiro's unified-harness documentation names four front ends to the same agent runtime — **IDE, CLI, Web, Mobile** — sharing `.kiro/` project config, with surface-specific behavior only for permissions and primary-agent selection. Specs can start in the IDE, continue in CLI, hand off to Web.

| Surface | What it is |
|---|---|
| IDE | Code OSS desktop editor: chat, specs, hooks UI, MCP panel, steering generation |
| CLI (`kiro-cli`) | Terminal-native agent: headless mode, session management, CI integration; V3 current |
| Web | Browser agent for multi-repo tasks — plans, implements, opens PRs; zero-setup sandboxed sessions |
| Mobile | Monitor tasks, review PRs, chat |

## Crew — separate orchestrator (not a harness front end)

- **✅** Crew is a personal-agent orchestration layer (autonomous tasks, scheduling, memory, multi-channel access) with its own gateway/configuration. It is **not** a front end to the unified Kiro harness: Crew 0.6 release notes show it selects among agent harness backends — `kiro-cli` (first-class), plus adapted Claude Code, Codex, OpenCode, KAS, Pi, and goose.
- **⚠️** Do not assume `.kiro/` config or the capability matrix below applies inside Crew-driven sessions.

## Capability matrix (from official docs)

| Capability | IDE | CLI | Web | Mobile |
|---|---|---|---|---|
| Workspace steering `.kiro/steering/` | ✓ | ✓ | ✓ | ✓ |
| Global steering `~/.kiro/steering/` | ✓ | ✓ | — | — |
| Inclusion modes | ✓ | V3 full; V1/V2 `always` only | ✓ | ✓ |
| AGENTS.md | ✓ | ✓ | ✓ | ✓ |
| MCP local + remote | ✓ | ✓ | ✓ | — |
| Hooks | ✓ | ✓ | ✓ | — |
| `.kiroignore` | ✓ | ✓ (V3) | — | — |

## kiro-cli notes

- **✅** `kiro-cli` is the terminal surface; docs track versions V1/V2/V3 — V3 adds full steering inclusion modes and the new hooks format.
- **✅** Migration path exists from Amazon Q Developer CLI ("Upgrading from Q CLI" doc) — `kiro-cli agent migrate` converts 2.x embedded hooks to `.kiro/hooks/*.json`.
- Headless + session management + CI integration are the differentiators vs the IDE surface.

## ACP

- **✅** ACP (Agent Client Protocol) integrations let other editors/clients drive the same Kiro agent.

## Enterprise

- **✅** Enterprise layer: SSO, governance, usage monitoring, team management (listed capability; details not verified for this entry).
