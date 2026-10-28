# Meta Muse Code agent harness

**Verified:** 2026-10-29 against current Meta developer documentation  
**Status:** Reference documentation, not a runtime conformance audit  
**Primary scope:** `muse` interactive CLI, headless/CI, server, schema, and SDK surfaces  
**Provenance:** Official `dev.meta.ai` Muse Code documentation unless explicitly marked otherwise

Muse Code has a fail-closed local sandbox, trusted-workspace instruction loading, layered permissions, and explicit subagent workspace choices. Organization-specific notes such as worktree territory are not vendor configuration paths and are not used as product evidence here.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration and context](configuration-and-context.md) | Settings path, trust, instruction-file selection, precedence, project memory |
| [Permissions and runtime](permissions-and-runtime.md) | Permission profiles, approval modes, sandbox, protected paths, network modes, subagents |
| [Agents, workflows, and extensibility](agents-workflows-and-extensibility.md) | Agent-tree limits, worktree isolation, hooks, MCP, saved workflows |
| [CLI, CI, and SDK](cli-ci-and-sdk.md) | Interactive/headless commands, server/schema, SDK, CI authentication and exit semantics |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“The executable is `muse-code`.”** Current official documentation uses `muse`.
- ❌ **“Muse Code uses `MUSE_CODE.md` as its project instruction file.”** Current documentation selects among `AGENTS.md`, `CLAUDE.md`, `.agents/AGENTS.md`, and `.claude/CLAUDE.md`.
- ❌ **“Muse Code supports only macOS and Linux.”** Current documentation includes Windows.
- ❌ **“Disabling the sandbox only relaxes command execution.”** It also removes file-tool workspace confinement and forces full network egress.
- ❌ **“`muse exec` exit code 0 proves the requested task and tests succeeded.”** It means the turn completed.

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Executable | `muse` |
| Interactive | `muse` |
| Headless | `muse exec` |
| Programmatic server/schema | `muse serve`, `muse schema` |
| SDK | `@muse-code/sdk` |
| CI authentication | `META_API_KEY` |
| User settings | `~/.config/muse/settings.json` |
| Project initialization | `muse init` writes only `AGENTS.md` |
| Instruction selection | `AGENTS.md` → `CLAUDE.md` → `.agents/AGENTS.md` → `.claude/CLAUDE.md` per directory |
| Shared project memory | `.agents/memory/MEMORY.md` plus topic files |
| Project hooks | `.muse/hooks.json` |
| MCP | `mcp_servers` in settings |
| Saved project workflows | `.agents/workflows/*.js` |
| Default new-session profile | Auto-review |
| Default network mode | `proxy-only` |

## Official sources

- [Muse Code overview](https://dev.meta.ai/docs/muse-code)
- [Configuration](https://dev.meta.ai/docs/muse-code/configuration)
- [Permissions](https://dev.meta.ai/docs/muse-code/permissions)
- [Extending Muse Code](https://dev.meta.ai/docs/muse-code/extending)
- [Workflows](https://dev.meta.ai/docs/muse-code/workflows)
- [Models](https://dev.meta.ai/docs/models)
