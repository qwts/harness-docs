# Agents, subagents, and skills

**Verified:** 2026-10-29

## Custom subagents

Project custom subagents use:

```text
.cursor/agents/
```

Cursor also documents compatibility discovery for `.claude/agents/` and `.codex/agents/`, with corresponding user-level directories. Compatibility means the files can be discovered; it does not prove identical frontmatter semantics or runtime behavior across vendors.

Subagents run with isolated context and can execute in foreground or background modes. Nesting is bounded rather than unbounded. Use a subagent for a separable workstream, not as a mechanism to duplicate the parent's entire conversation and configuration indefinitely.

### Cloud boundary

Cloud subagents use team/cloud MCP configuration. They do not inherit the parent agent's local MCP configuration merely because the parent launched them. Credentials and tools available to a local parent therefore cannot be assumed available to a cloud child.

## Skills

Skills are discovered from project and user skill directories, including:

```text
.agents/skills/
.cursor/skills/
```

Cursor also documents user equivalents and Claude/Codex compatibility directories. A skill is a directory with a `SKILL.md` manifest and optional supporting files.

Required frontmatter:

```yaml
---
name: example-skill
description: What the skill does and when the agent should use it.
---
```

Documented optional fields include:

- `paths`;
- `disable-model-invocation`;
- `icon`;
- `color`;
- `metadata`.

The description is selection metadata, not decorative text. It should identify both the capability and its intended trigger conditions.

## Personal skill sync and cloud execution

Only `~/.cursor/skills/` participates in personal skill sync for Cloud Agents. A skill present in another local compatibility directory can work locally without becoming a synced personal cloud skill.

Automation runs do not load synced personal skills. Therefore these statements are different:

- “The skill is available to local Cursor.”
- “The skill syncs to Cloud Agents.”
- “The skill is available during an automation run.”

Only the first two can follow from placement in the documented local/sync directories; the third is not implied.

## Sources

- [Subagents](https://cursor.com/docs/subagents)
- [Skills](https://cursor.com/docs/skills)
- [Cloud Agent setup](https://cursor.com/docs/cloud-agent/setup)
