# Agents, skills, and automation

**Verified:** 2026-10-29

## Skills

The recommended project skill location is `.cline/skills/<skill>/SKILL.md`. Cline also documents compatibility locations, including `.clinerules/skills/`, `.claude/skills/`, and `.agents/skills/` through shared configuration support.

| Scope | Locations |
|---|---|
| Project | `.cline/skills/` (recommended), plus documented compatibility paths |
| Global | `~/.cline/skills/`, `~/.agents/skills/` |

**Precedence is counterintuitive:** a same-named global skill overrides a project skill. Name collisions therefore cannot be used to assume that repository-local instructions win.

### `SKILL.md` contract

```markdown
---
name: release-checks
description: Run and interpret this repository's release validation.
---

Follow the repository release checklist and report failed checks.
```

- `name` and `description` are required.
- `name` must exactly match the skill directory name.
- `description` has a 1,024-character maximum.
- Cline recommends keeping the `SKILL.md` body under 5,000 tokens.

Skills use progressive loading: metadata is available at startup, instructions load when triggered, and supporting resources load as needed. They can be invoked automatically or through slash commands. This reduces context use; it does not make skill resources or instructions trusted merely because they were loaded lazily.

## Subagents

Subagents are documented for **Cline CLI and Desktop**, not for the VS Code or JetBrains extensions.

| Property | Documented behavior |
|---|---|
| Context | Separate context window from the parent |
| Execution | Can run in parallel |
| Nesting | Can spawn further subagents |
| MCP | Subagents do not receive MCP tools |
| Approval | Approving `spawn_agent` approves the child’s subsequent tool activity |

The approval behavior is security-relevant: child tool calls do not independently prompt after `spawn_agent` is approved. Review the requested child task and its likely capabilities before approving creation; do not expect a second approval boundary for every child action.

Plan mode can restrict editing and some shell activity, but it is not a non-write guarantee. It should not be used as the sole containment mechanism for a subagent.

## Agent teams

Agent teams are a separate system from subagents. They are documented for **SDK, CLI, and Kanban**, not the VS Code or JetBrains extensions.

Teams use:

- a coordinator;
- a shared task board;
- a mailbox for inter-agent messages;
- a mission log.

Team state is stored in:

```text
~/.cline/data/db/teams.db
```

Teams can be started with `/team`. They are unavailable with `--yolo`, so a deployment cannot simultaneously rely on team orchestration and that bypass mode.

### Choosing between subagents and teams

- Use **subagents** for delegated work with separate context, including parallel or nested delegation.
- Use **agent teams** when the work needs a coordinator and shared coordination state across participants.
- Do not infer MCP access, independent child approvals, or IDE-extension availability for either feature.

## Operational cautions

Skills, delegated agents, and team messages can all influence tool-using execution. Review repository-provided skills and instructions, isolate secrets, and validate outputs independently. Delegation changes context and coordination; it does not by itself establish trust or a sandbox.

## Sources

- [Skills](https://docs.cline.bot/customization/skills)
- [Configuration](https://docs.cline.bot/getting-started/config)
- [Subagents](https://docs.cline.bot/features/subagents)
- [Agent teams](https://docs.cline.bot/cli/agent-teams)
- [Plan and Act](https://docs.cline.bot/core-workflows/plan-and-act)
