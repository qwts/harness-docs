# Agents, skills, and MCP

**Verified:** 2026-11-15

## Subagents

“A subagent is a separate worker Command Code hands a task to. It runs in its own context window with its own tools, system prompt, and (optionally) its own model,” and several can run concurrently. Built-ins: General (the default delegate target), Explore (read-only), and Plan (read_file only). You never invoke one directly — “Command Code decides when to delegate by calling its built-in agent tool,” matching against the agent’s `description`.

Agents are Markdown files — “The front matter configures it; the body is the system prompt” — with exactly these read keys: `name` (required, sanitized, reserved names `explore/plan/review/general` are ignored), `description , `tools` (`"*"` grants all; omitted means none; `agent`/`agent_output` can’t be granted), `disallowedTools  (deny beats allow), `model` (any `/model` id, or inherit), `reasoningEffort`, `maxTurns` (default from `subagentMaxTurns`, else 200; range 1–1000), `permissionMode`, `background`, `showOutput`. Locations: `.commandcode/agents/` (project, committed) then `~/.commandcode/agents/` (personal) — “the first definition of a name wins,” re-scanned each turn. “Subagents can’t start their own subagents - the agent tool is removed from their tool set.”

`background: true` returns an `agent_id` immediately; results arrive via the `agent_output  tool, and Ctrl+B moves a foreground agent to background mid-run. Permission-wise, sub-agents auto-allow where the main loop would prompt, but “hard guards” (deny rules, plan mode, safety prompts) … "fail closed."

## Skills

“Agent Skills are modular, self-contained instruction sets” — a folder with a `SKILL.md` (frontmatter `name` max 64 chars and `description` max 1024 required; optional `allowed-tools`, `model`, `effort`, `argument-hint`, and more), loaded with progressive disclosure: only name/description/path at startup, full body on activation. Storage: `~/.commandcode/skills/` (user), `.commandcode/skills/` (project, “should be shared with your teamn”), plus compatibility roots `.agents/skills/` (project) and `~/.agents/skills/` (user) — “`.commandcode/skills/` takes priority on name conflicts.” There is no `.claude/skills/` root in the documented list. Priority: project `.commandcode/ → project `.agents/` → user `~/.commandcode/` ↲ user `~/.agents/` → extra `skills` paths in settings → bundled skills; `/skill:<name>` disambiguates collisions with built-ins.

Skill bodies can embed live shell output (`!`...`` inline or fenced `!` blocks) — “so opening a skill never shells out” is the explicit opt-out, `disableSkillShellExecution: true`, which replaces every placeholder with “[shell command execution disabled by policy].” Placeholders include `$ARGQMENTS`, `${COMMANDCODE_SKILL_DIR}`, `${COMMANDCODE_PROJECT_DIR}` (with `CLAUDE_*` aliases). Manage with `cmd skills add <owner/repo>` / `list` / `remove`; per-invocation: exact `/skill-name <args>`, inline mid-prompt, or inferred suggestion. Skills are authored playbooks — “Taste is the implicit, continuously-learned profile.”

## MCP

Three scopes, with the docs’ own precedence: “When the same server name exists in multiple scopes, local overrides project, which overrides user.”

| Scope | File |
|---|---|
| local (default) | `~/.commandcode/projects/<slug>/mcp.json` — “Private, per-project” |
| project | `.mcp.json` at project root — “Shared, version controlled” |
| user | `~/.commandcode/mcp.json` — “Private, available across all projects” |

The settings reference lists the same ladder including `settings.json` `mcp.servers` below user `mcp.json`. Transports: stdio (`cmd mcp add my-tool -- npx @my-org/mcp-server`) and http (`\�--transport http <name> <url>`); flags precede the name. Env interpolation supports `${VAR}`, `${VAR:-default}`, and literal `$${VAR}`; an unset variable without a default “stops that server with an error naming the variable.” `/mcp` opens an in-session manager with connection status; OAuth tokens live in `~/.commandcode/mcp-tokens.json` (auto-refreshed; OAuth secrets are stripped from project `.mcp.json`). MCP tools surface as `mcp__<server>__<tool>` — e.g. `mcp__github__get_issue` — which is also the permission-rule name. Headless time-bounds come from `MCP_TOOL_TIMEOUT` and `MAX_MCP_OUTPUT_TOKENS` (default 25000).

## Official sources

- [Agents](https://commandcode.ai/docs/agents)
- [Skills](https://commandcode.ai/docs/skills)
- [MCP](https://commandcode.ai/docs/mcp)
- [Background Tasks](https://commandcode.ai/docs/background-tasks)
- [Settings & Config](https://commandcode.ai/docs/settings)
