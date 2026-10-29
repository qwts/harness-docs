# Agents, skills, and recipes

**Verified:** 2026-10-29

## Choose the right reusable unit

| Mechanism | What it packages | Execution model |
|---|---|---|
| Custom agent | Role, instructions, description, optional model preference | Load into the current conversation or delegate to a separate session |
| Skill | On-demand procedure/domain knowledge and supporting files | Discovered at startup; full instructions loaded when selected or requested |
| Recipe | Instructions/prompt, parameters, extensions, model settings, output and retry behavior | Reusable interactive, headless, delegated, or scheduled workflow |
| Subagent | A scoped task and optional recipe/configuration | Temporary independent goose execution returning results to the parent |
| Subrecipe | A recipe registered by another recipe | Separate session and context; experimental; no nested subrecipes |
| Custom slash command | A name mapped to a recipe | Loads that recipe in a chat session |

These mechanisms compose, but an agent file is not a workflow, a skill is not an MCP server, and a recipe is not just a saved prompt.

## Custom agents

Recommended locations:

- Global: `~/.agents/agents/`
- Project: `<project>/.agents/agents/`

Compatibility discovery also covers documented `.goose/agents/`, `.claude/agents/`, corresponding home directories, and goose’s platform config directory. New shared agents should use `.agents/agents/`.

Each Markdown file has YAML frontmatter. `name` is required; `description` and `model` are optional. The body supplies instructions.

Two uses are materially different:

- **Load** adds the agent’s instructions to the current conversation.
- **Delegate** runs a separate session and returns a result; delegation can use the agent’s model preference and interface-supported overrides.

An agent file does not define its own MCP list, schedule, or step workflow. It uses skills, delegation tools, and extensions available in the session. Use a recipe when those must be packaged reproducibly.

## Subagents

Internal subagents run temporary goose instances for sequential or parallel work. They preserve the parent’s conversational focus and isolate process/context lifecycle; this is not, by itself, filesystem or network confinement.

Key behavior:

- Goose can autonomously create subagents only in the default autonomous mode. They are disabled in Manual Approval, Smart Approval, and Chat Only modes.
- Internal subagents inherit parent extensions by default; prompts or recipes can narrow the set.
- Default concurrency is five per parent session; extra tasks wait.
- Default timeout is five minutes. A failed or timed-out subagent returns no output; parallel execution returns only successful results.
- Subagents cannot create more subagents, manage extensions, or manage schedules.
- `delegate` and `load` come from the `summon` platform extension, enabled by default. A recipe with an explicit extension list must include `summon` to delegate, unless it declares `sub_recipes`, which auto-injects it.

Natural-language customization is task configuration, not an administrative security control. Model/provider and turn-limit precedence are documented in [Configuration, precedence, and providers](configuration-precedence-and-providers.md).

External subagents can be separate agents exposed as MCP servers. Their own approval and sandbox settings belong to that external agent and must not be attributed to goose core.

## Skills

Recommended discovery roots:

1. `~/.agents/skills/` for global skills.
2. `<project>/.agents/skills/` for project skills.
3. `~/.agents/plugins/<plugin-name>/` for plugin-provided skills.

A skill is a named directory containing `SKILL.md` with required `name` and `description` frontmatter. Goose places discovered names/descriptions in startup instructions and loads the full skill when the request matches or the user explicitly selects it. `goose skills list` and the interactive `/skills` command provide direct discovery/loading.

Open Plugin skill names are namespaced as `plugin:skill`; Gemini-extension skills retain their own names. Compatibility paths include `.goose/skills/`, `.claude/skills/`, `~/.claude/skills/`, and platform config directories.

Skills may include scripts, templates, and other resources. Loading a skill exposes instructions and supporting-file knowledge; it does not automatically execute every resource. If the agent later runs a script through Developer or another tool, that execution has the tool’s actual privileges and approval behavior. Review supporting files as executable artifacts, not harmless prose.

## Recipe schema and settings

Recipes use YAML or JSON and require `title`, `description`, and at least one of `instructions` or `prompt`. Optional fields cover:

- `activities` for Desktop prompt bubbles;
- `extensions` and per-extension `available_tools`;
- typed `parameters` with required/optional/user-prompt behavior;
- `settings` for provider, model, temperature, and turns;
- `response.json_schema` for validated structured output;
- `retry` with shell success checks and optional cleanup;
- `sub_recipes` for specialized child workflows.

Desktop accepts `.yaml`, `.yml`, and `.json`; the CLI supports `.yaml` and `.json`, not `.yml`. Desktop wraps saved recipes in library metadata that the CLI ignores. `activities` and that library metadata do not change CLI/scheduled execution.

An explicit recipe `extensions` block limits the recipe to the listed extensions; defaults are not implicitly merged in. Missing extension secrets are not requested during recipe load and must be configured beforehand.

### Storage and discovery differ by interface

| Source | Behavior |
|---|---|
| `~/.config/goose/recipes/` | Global recipe library |
| `<project>/.goose/recipes/` | Project-local recipes |
| Current directory and `GOOSE_RECIPE_PATH` | CLI recipe discovery |
| Configured GitHub repository | CLI fallback/discovery using authenticated GitHub CLI |
| Desktop Recipe Library | Shows explicitly saved or imported recipes, not every discoverable filesystem file |

Desktop provides visual create/edit/import/export and deeplinks. The CLI provides `goose recipe list`, `validate`, `deeplink`, `open`, and `goose run --recipe`. First-time Desktop execution displays recipe details for trust review and warns again if the recipe changes; this review does not make its commands safe.

## Subrecipes

Subrecipes are experimental. The parent registers a name, path, and optional fixed values. Goose exposes each as a tool, executes it in a separate session, and returns its output.

Subrecipe sessions do not share conversation history, memory, or state with the parent or siblings. They cannot declare their own subrecipes. Fixed `values` override context-derived parameters. Separate context does not erase external side effects from tools or shell checks.

## Retry and structured output are executable automation

Recipe retries run every configured shell success check in order. A failed check can run `on_failure`, reset the agent’s message history, and start another attempt. Resetting messages does not roll back files, processes, network calls, or other effects. Recipe-specific check/cleanup timeouts override global retry timeout variables.

Structured responses call a `final_output` tool and validate the result against the supplied JSON schema. This validates response shape, not whether the task’s factual claims or side effects are correct.

## Schedules and slash commands

Desktop schedules a recipe in background mode or foreground mode (opening a window if Desktop is running, otherwise background). The CLI uses `goose schedule add` with 5-, 6-, or 7-field cron expressions. Schedule details link to created sessions. These are local goose automation surfaces, not proof of a vendor-hosted scheduler or a security-isolated worker.

Custom slash commands map a command name to a recipe. They accept one parameter; other parameters need defaults. Names are case-insensitive, unique, space-free, and cannot collide with built-ins. If the mapped recipe is missing or invalid, the input is treated as ordinary model text rather than producing a reliable command error.

## Trust checklist

Before running a shared recipe, agent, skill, or schedule, inspect:

- Instructions and prompt-template expansion.
- Stdio extension commands and remote endpoints.
- Required credentials and their scope.
- Retry checks and cleanup shell commands.
- Supporting scripts and files.
- Subagent/subrecipe extension inheritance.
- Schedule frequency, working directory, and external side effects.

## Official sources

- [Custom agents](https://goose-docs.ai/docs/guides/context-engineering/custom-agents)
- [Subagents](https://goose-docs.ai/docs/guides/context-engineering/subagents)
- [Skills](https://goose-docs.ai/docs/guides/context-engineering/using-skills)
- [Recipe reference](https://goose-docs.ai/docs/guides/recipes/recipe-reference)
- [Reusable recipes](https://goose-docs.ai/docs/guides/recipes/session-recipes)
- [Saving and discovering recipes](https://goose-docs.ai/docs/guides/recipes/storing-recipes)
- [Subrecipes](https://goose-docs.ai/docs/guides/recipes/subrecipes)
- [Custom slash commands](https://goose-docs.ai/docs/guides/context-engineering/slash-commands)
