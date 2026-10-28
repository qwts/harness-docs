# Instructions, agents, and skills

**Verified:** 2026-10-29 · **Primary surface:** Copilot CLI

## Instruction discovery

| Type | Location |
|---|---|
| User baseline | `~/.copilot/copilot-instructions.md` |
| Modular user | `~/.copilot/instructions/**/*.instructions.md` |
| Repository baseline | `.github/copilot-instructions.md` |
| Modular repository | `.github/instructions/**/*.instructions.md` |
| Agent instruction formats | `AGENTS.md`, `CLAUDE.md`, `.claude/CLAUDE.md`, `GEMINI.md` |
| Additional directories | Comma-separated `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` |

Unless the documentation states otherwise, repository and agent files are discovered at the repository root, current working directory, intermediate directories, and nested directories along a file being edited. Modular `*.instructions.md` files use `applyTo` globs and apply only to matching files.

All applicable instructions are combined. Identical copies of some baseline and agent instruction files are deduplicated, but GitHub defines no general precedence order. Conflicting instructions are therefore an authoring error, not a reliable override mechanism.

Use `/instructions` to inspect, enable, or disable discovered files. Changes are not immediately applied to an active session; restart/resume or start a new session.

### File references

`@relative/path` includes are supported in `.github/copilot-instructions.md`, `AGENTS.md`, and `CLAUDE.md`, including nested references. Targets must stay within the repository or applicable custom-instruction directory. Absolute paths and `~/` targets are not loaded. These includes are not expanded in `GEMINI.md` or modular `*.instructions.md` files.

## Custom agents and subagents

Custom agent profiles are Markdown files with YAML frontmatter, normally using `.agent.md`:

```text
Project: .github/agents/
User:    ~/.copilot/agents/
```

Organization and enterprise profiles also exist. In the CLI, a user profile overrides a repository profile with the same ID; repository overrides organization, and organization overrides enterprise. The ID comes from the filename, not the optional display `name`.

The Markdown prompt is limited to 30,000 characters. Tool filtering has three materially different states:

| `tools` value | Meaning |
|---|---|
| Omitted or `["*"]` | All available tools |
| Explicit list | Only listed built-in or MCP tools |
| `[]` | No tools |

Unrecognized tool names are ignored, which permits cross-surface profiles but can also hide typos.

A custom agent selected as the session agent receives repository custom instructions. The same agent spawned as a subagent does not, by default. Opt in per profile:

```yaml
---
name: RepoAwareReviewer
description: Reviews changes against repository conventions
tools: ["*"]
include-custom-instructions: true
---
```

This opt-in does not import extra directories supplied only to the parent through `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`. `--no-custom-instructions` still wins globally.

## Skills

Skills are directories containing `SKILL.md` plus optional scripts and resources.

| Scope | Locations |
|---|---|
| Project | `.github/skills/`, `.claude/skills/`, `.agents/skills/` |
| Personal | `~/.copilot/skills/`, `~/.agents/skills/` |

Required frontmatter fields are `name` and `description`; `license` is optional. Optional `allowed-tools` can pre-approve tools, but pre-approving `shell` or `bash` removes an important confirmation boundary and should be limited to reviewed, trusted skills.

Copilot can select a skill from its description, or the user can invoke `/skill-name`. `/skills list`, `/skills info`, `/skills reload`, and the `copilot skill` terminal subcommand support inspection and management.

## Sources

- [Adding custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Creating and using custom agents](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [Adding agent skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
