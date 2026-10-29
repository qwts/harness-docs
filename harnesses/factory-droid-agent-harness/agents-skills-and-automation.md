# Agents, skills, and automation

**Verified:** 2026-10-29

## Custom droids are fresh-context subagents

A custom droid is a Markdown-defined subagent with its own system prompt, model preference, and tool policy. Every new invocation runs through the `Task` tool in a fresh context and returns one final message to the parent.

Definition locations are:

```text
<repo>/.factory/droids/<name>.md
~/.factory/droids/<name>.md
```

Only the top level of each `droids/` folder is scanned. A project definition wins over a personal definition with the same name. Built-in `worker` and `explorer` droids are available without custom files.

```markdown
---
name: code-reviewer
description: Focused reviewer that checks diffs for correctness risks
model: inherit
tools: read-only
---

You are the team's senior reviewer. Examine the diff and report correctness,
security, migration, and testing risks.
```

Tool-policy details matter:

- omit `tools` to allow all tools; literal `tools: all` is rejected;
- use a category such as `read-only` or an array of exact tool IDs for restrictions;
- `TodoWrite` and `Skill` are always included;
- `ExitSpecMode` and `GenerateDroid` cannot be enabled by a custom droid;
- omitting `mcpServers` inherits parent MCP availability, while `mcpServers: []` excludes every MCP server;
- organization tool, model, autonomy, and MCP restrictions still apply.

Subagents are non-interactive: `AskUser` is disabled and a subagent cannot spawn another subagent. A blocked or ambiguous child reports back rather than asking the user directly.

`Task` supports foreground and background execution. Background work returns a task ID, with `TaskOutput` for retrieval and `TaskStop` for termination. A later `resume` continues an existing subagent session with its prior context; fresh-context invocation and resumed execution are therefore different operations.

Model selection starts with a droid-pinned model, then complexity routing, then the parent model. If a pinned model is disallowed or unavailable, validation falls back to the parent model rather than running the prohibited selection.

Autonomy defaults to `inherit`, can be set separately for subagents, and is always clamped by the organization maximum. In Spec Mode, subagents are read-only. Tool restrictions and autonomy are complementary: a read-only tool policy limits available tools, while autonomy governs approval for tools that remain available.

## Skills are progressive-disclosure instructions

A skill is a directory whose entry point is exactly `SKILL.md`, with optional supporting files. Metadata is discovered first; the full body loads when selected or invoked.

```text
.factory/skills/review-api-change/
  SKILL.md
  checklists.md
  scripts/
    verify-api-change.sh
```

Documented sources include project, folder-specific, personal, compatibility, mission, automation, plugin, and built-in scopes. Droid searches skill folders recursively; the first directory containing `SKILL.md` becomes the skill directory.

Outside mission and automation sessions, ordinary same-name precedence is:

```text
Folder/project
Project plugins
Personal
User plugins
Built-in
```

Organization plugin skills outrank all of those sources, including mission and automation skills. Mission skills outrank project skills within a mission. Automation skills override other ordinary sources in that automation. Use `/skills` and its Effective view to determine the actual winner rather than inferring it from one file path.

`disabledSkills` arrays from user and project settings combine, so disabling a name at either level remains effective. Frontmatter provides separate controls:

```yaml
---
name: review-api-change
description: Review API changes for compatibility and migration risks.
enabled: true
user-invocable: true
disable-model-invocation: false
---
```

- `enabled: false` disables both user and model invocation;
- `user-invocable: false` removes slash invocation but leaves model invocation available;
- `disable-model-invocation: true` prevents automatic selection but leaves direct user invocation available.

`allowed-tools` declares intended tools for review and display. It neither grants unavailable tools nor sandboxes standard skill invocation. Use a custom droid with an enforced tool policy when the workflow requires an actual runtime restriction.

Skill scripts and repository-provided skill instructions remain trusted inputs to a tool-using session. Progressive disclosure reduces context use; it does not make the content safe.

## Custom slash commands

Custom commands are discovered recursively from:

| Scope | Location | Collision behavior |
|---|---|---|
| Workspace | `<repo>/.factory/commands/` | Overrides a personal command with the same slug |
| Personal | `~/.factory/commands/` | Used when no workspace command wins |

A file is registered if it ends in `.md` or starts with a shebang. Markdown commands support `$ARGUMENTS`, not `$1` or `$2`. They provide no tool-scoping boundary.

Executable commands receive positional arguments, run from the current working directory, and inherit the environment. Up to 64 KB of stdout/stderr plus the script contents are posted back to the transcript. That transparency can expose command output or script-embedded sensitive material, so review both before running commands in sessions that synchronize or share transcripts.

Factory recommends skills for new reusable workflows; existing custom commands remain supported.

## Missions

Mission Mode upgrades a session into an orchestrator workflow with worker agents, validation, and progress tracking. It is separate from one-off custom-droid delegation and from Spec Mode planning.

Missions can inherit the surrounding Droid configuration, including MCP integrations, custom skills, hooks, custom droids, and `AGENTS.md`. Workers can therefore receive substantial capabilities; Mission orchestration is not itself an isolation mechanism.

Headless execution is available:

```sh
droid exec --mission -f mission.md
```

Model and reasoning defaults can be configured for the orchestrator, ordinary workers, and validation workers. The six mission model/reasoning fields resolve independently using session-default precedence. Organization `missionPolicy` can restrict which users may start missions. Missions require High autonomy or `--skip-permissions-unsafe`, and organization access controls still apply.

## Automations

Automations start a Droid session whenever a trigger fires. Every automation combines:

```text
trigger + instructions + run identity + compute location
```

Documented trigger types are:

| Trigger | Start condition | Execution note |
|---|---|---|
| Scheduled | A cron-style schedule comes due | Runs the configured instructions on the selected compute |
| Slack | A message arrives in a selected channel | Message content becomes an input to an automated tool-using run |
| GitHub | A repository event such as a pull request or push fires | Runs in GitHub Actions |
| Webhook | Another service sends an HTTP POST to the automation URL | Available when enabled for the organization |

Review trigger inputs, run identity, repository and connector permissions, instructions, and compute together. A scheduled or event-driven run has no interactive user present to supply the judgment that an interactive approval workflow might otherwise provide. Use least-privilege identities and deterministic validation for side effects.

Automations and Delegations are different surfaces: Automations are trigger-defined jobs, while Delegations hand work to Droid from supported collaboration and tracker applications.

## Sources

- [Custom droids](https://docs.factory.com/harness/subagents)
- [Skills](https://docs.factory.com/harness/skills)
- [Custom slash commands](https://docs.factory.com/harness/custom-slash-commands)
- [Missions configuration and reference](https://docs.factory.com/missions/reference)
- [Automations](https://docs.factory.com/automations/overview)
- [Custom Automations](https://docs.factory.com/automations/custom)
- [Autonomy Level](https://docs.factory.com/autonomy-and-safety/auto-run)
