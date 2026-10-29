# Agents, skills, and automation

**Verified:** 2026-10-29

## Filesystem skills

Local skills can be directory bundles or flat Markdown files:

```text
<root>/<name>/SKILL.md
<root>/<name>.md
```

Nested `**/SKILL.md` discovery is deliberately unsupported. Required YAML frontmatter fields are `name` and `description`; `name` must be kebab-case. Optional fields include `whenToUse`, `metadata`, `disable-model-invocation`, and `user-invocable`.

- `disable-model-invocation: true` removes the skill from model-facing catalogs and loaders.
- `user-invocable: false` removes it from human-facing commands.
- Invalid invocation values drop the whole skill with a warning rather than silently enabling a surface.

### Discovery priority

Default roots are scanned in this rank order:

| Rank | Root |
|---:|---|
| 100 | `<projectRoot>/.dsh/skills` |
| 200 | `<projectRoot>/.agents/skills` |
| 300 | `customSkillDirs` |
| 400 | `$DSH_HOME/skills` |
| 500 | `$DSH_AGENTS_HOME/skills` |
| 600 | Configured bundled skill directory |

Project roots therefore have higher provider priority than user roots. The project root is the nearest `.git` ancestor, or the current working directory when no marker exists. `includeDefaultRoots: false` restricts discovery to explicitly configured roots.

Discovery parses frontmatter into a catalog; loading a skill re-reads its current body. Watchers refresh catalogs for skill additions, removals, renames, and frontmatter changes. Resource-subtree edits do not trigger the same catalog refresh, and an already-loaded body is not rewritten in retained history.

Malformed skills are skipped with a warning. The model catalog cannot distinguish an invalid skill from an absent one, so audit logs when an expected skill does not appear.

## Subagents are provider-based

Subagent support requires three composed pieces:

1. the `dsh-subagent` service;
2. at least one provider backend;
3. a model-facing delegation tool.

Mounting the service alone enables no delegation. Local spawn/fork children retain durable sessions and accept later messages; ACP, DSH SDK, Codex, and Claude Code external children execute once and reject continuation.

Children inherit parent origin metadata and permission roots. In-process children receive a fixed delegated scope and cannot widen it from inside the child session; operations requiring approval are rejected automatically.

### Depth and capacity

- Default `maxDepth` is `1`: direct children only.
- Depth `0` disables delegation through tools inheriting that setting.
- An explicit tool depth overrides the host setting.
- Default `maxActiveSubagents` is `8` per activation tree.
- Admission at capacity rejects rather than queues.
- Capacity is process-local and does not bound cumulative session history or token use.

Codex and Claude Code providers are separate optional bundles. Installing one registers its dormant provider on the next profile start; a copied preset must separately enable the corresponding tool row for new agents.

## Agent teams are experimental

Agent Teams are an optional experimental capability, not the default subagent mechanism. The documented bundle provides named teammates, direct messages, and a durable shared task board. Experimental status means the capability is under evaluation, has no product-support commitment, and may change or be withdrawn.

Do not infer that subagent depth/capacity settings, external-provider behavior, or hook support automatically apply to agent teams.

## Scheduling and unattended work

Scheduling is plugin/profile-dependent rather than evidence that every surface has a durable background worker. Desktop does not enable scheduled tasks by default; reminders fire only in loaded sessions, and quitting can prevent them from running.

For unattended execution, verify the selected profile, session persistence, permission mode, approval behavior, and host lifetime. A persisted session or scheduled reminder does not prove that a process will remain alive to execute it.

## Sources

- [Filesystem skills](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/skill-filesystem/README.md)
- [Subagent service](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/subagent/subagent/README.md)
- [CLI plugin management](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
- [Experimental package status](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/experimental/README.md)
- [Desktop reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.md)
