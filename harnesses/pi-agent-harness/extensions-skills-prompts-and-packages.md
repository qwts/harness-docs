# Extensions, skills, prompts, and packages

**Verified:** 2026-11-16 (Pi v1.5.2)

## Extensions

“Extensions are TypeScript modules that extend Pi's behavior. They can subscribe to lifecycle events, register custom tools callable by the LLM, add commands, and more.” They load via jiti (“so TypeScript works without compilation”) into the Pi process: “An extension runs inside the Pi process with the same operating-system permissions. It can inspect prompts, tool calls, files, credentials, and session history, so load extensions only from sources you trust.”

Locations: `~/.pi/agent/extensions/*.ts` or `*.js` (global), subdirectory `index.ts`/`index.js`, subdirectory `package.json` with a `pi.extensions` field, and project `.pi/extensions/…` (“Project-local `.pi/extensions` entries load only after the project is trusted”). “Discovery goes one level deep. Larger packages declare their entry points in a `package.json` manifest.” Settings add paths under `extensions` and `packages`; `--extension`/`-e` loads one for a run (“Use `pi -e ./path.ts` only for quick tests”), and `-ne/--no-extensions` disables discovered, configured, and built-in extensions while “Explicit `-e` paths still load.”

The shape is a default factory function receiving `ExtensionAPI` (`pi`), optionally async — “If the factory returns a `Promise`, Pi awaits it before continuing startup.” Registration methods include `pi.registerTool()`, `pi.registerCommand()`, `pi.registerShortcut()`, `pi.registerFlag()`, `pi.registerProvider()`, `pi.registerVirtualModel()`, and `pi.registerMcpServer(name, config)`. Events cover the whole lifecycle: `project_trust`, `resources_discover`, `session_start`/`session_before_switch`/`session_before_fork`/`session_before_compact`/`session_compact`/`session_before_tree`/`session_shutdown`, `before_agent_start`, `agent_start`/`agent_end`/`agent_before_settle`/`agent_settled`, `turn_start`/`turn_end`, `message_start`/`message_update`/`message_end`, `tool_execution_*`, `tool_call`/`tool_result`, `context`/`context_with_system`, `before_provider_headers`/`before_provider_request`/`after_provider_response`/`provider_stream_event`, `model_select`, `thinking_level_select`, `user_bash`, and `input`. “Handler composition” and await rules are documented in the extensions reference.

Tool semantics worth noting: tools declare TypeBox (JSON Schema) parameters — “Pi validates tool arguments as JSON Schema … Before validation, loosely typed arguments are coerced: numeric strings to numbers, `"true"` to `true`, a single value to a one-item array” — with `promptSnippet`/`promptGuidelines` for system-prompt placement, `defaultActive: false` for inactive registration, `outputSchema`/`structuredContent` for programmatic callers, and `terminate: true` to end the run after a batch. **Exposure levels:** `direct` (default; “declared to the model while active, and callable while active”), `model-only` (“declared … never callable”), `codemode` (“callable whenever registered … Not declared to the model unless activated explicitly”), `deferred` (“like `codemode`, but codemode tools do not list it; `tool_search` can find and activate it”), and `hidden` (“registered but unreachable. Re-register a tool with `exposure: "hidden"` to withdraw it, since tools cannot be unregistered”). State belongs in tool-result `details` (“Rebuild from `getBranch()`, not from every entry”) or durable `pi.appendEntry()` custom entries.

Operational rules: “Extensions with state should store it in tool result `details` for proper branching support”; factories “may run in invocations that never start a session. Do not start background resources … from the factory” — defer to `session_start` and register an idempotent `session_shutdown` handler. `/reload` hot-reloads auto-discovered extensions.

## Skills

“Skills give Pi specialized instructions and supporting files for a particular kind of work. Pi advertises each available skill by name and description, then loads its full instructions only when the task calls for them. … Pi implements the Agent Skills specification.” A skill is “a directory containing `SKILL.md`” with frontmatter (`name`, `description`, plus `license`, `compatibility`, `metadata`, `allowed-tools` — “Experimental pre-approved tool list” — and `disable-model-invocation`). Constraints: “Names use lowercase letters, numbers, and hyphens … at most 64 characters; descriptions can contain at most 1024,” “Malformed `SKILL.md` files and declared skills without descriptions are not loaded,” and “Name collisions keep the first discovered skill and produce a warning.” Portability nuance: “Pi neither requires nor warns when the declared name differs from the parent directory.”

Loading: “At startup, Pi scans configured skill locations and adds each skill's name, description, and path to the system prompt. It does not add the full instructions. When a task matches, the model reads `SKILL.md` … A model might fail to load a relevant skill, so use `/skill:name` when you need to force it. Arguments after `/skill:name` are appended to the loaded instructions as a user request.” `disable-model-invocation: true` hides the skill from automatic selection; `enableSkillCommands` (default true) controls whether “skill commands appear in interactive command discovery; manually entered `/skill:name` commands still work.”

Locations: “Place the skill in your user or project skills directory. Directories containing `SKILL.md` are discovered recursively” — `~/.pi/agent/skills/`, `.pi/skills/` (trust-gated), plus the Agent Skills locations `~/.agents/skills/` and `.agents/skills/` (“Project `.agents/skills/` directories are discovered from the working directory through its ancestors, stopping at the repository root when one exists”). Trust note: “Project skills can instruct the model to run scripts or modify files. Review unfamiliar skills and their supporting files before granting project trust.”

## Prompt templates

“Prompt templates turn Markdown files into reusable `/` commands.” Create `~/.pi/agent/prompts/review.md` with `description` and optional `argument-hint` frontmatter; “The filename becomes the command name.” Substitutions: `$1`, `$2`, …; `$@` or `$ARGUMENTS`; `${1:-default}`; `${@:-default}`; `${@:N}`; `${@:N:L}` — “Arguments follow shell-like quoting, so `/review "API compatibility"` supplies one argument containing a space.” “Pi expands the template before the resulting text enters the agent. Extensions receive the raw input first through the `input` event unless an extension command with the same name handles it.” Conventional directories “load direct `.md` children only”; project templates “become commands in the editor after trust is granted.”

## Themes

Theme files (JSON) load from `~/.pi/agent/themes/` and `.pi/themes/`, or via `--theme <path>`; `--use-theme <name[/name]>` “Selects the initial interactive theme for this run,” and the `theme` setting defaults to `"system"` (“derives colors from the terminal theme”).

## Pi packages

“A package is an ordinary directory or npm package” that bundles extensions, skills, prompt templates, and themes:

```bash
pi install npm:@example/pi-tools@1.0.0
pi install git:github.com/example/pi-tools@v1
pi install ./local-package
```

“`pi list` shows configured packages. Use `pi remove <source>` to remove one and `pi update --extensions` to reconcile package installations.” Sources: npm (“Installed under the Pi npm directory”), git (“Cloned and reconciled to the selected ref”), URLs (“Treated as a git source”), and local paths (“Loaded from the resolved path without copying”). “Versioned npm specifications are pinned. Git tags and commits are also pinned; package updates reconcile the checkout but do not move a configured ref.” Personal installs write `~/.pi/agent/settings.json`; `--local` writes `.pi/settings.json`, and “Pi reads declarations from that file only after project trust is granted.” Failure behavior: “Pi warns about it, skips that package's resources, and keeps running.”

Manifest and gallery: conventional `extensions/`, `skills/`, `prompts/`, `themes/` directories work without a manifest; the `pi` key selects explicit paths with globs and exclusions. “The `pi-package` keyword makes an npm package eligible for discovery in the Pi package gallery.” Changelogs: “On the next fresh launch, Pi reads the package changelog and shows entries newer than the last seen version” under “Extension Updates” (default `CHANGELOG.md`, `pi.changelogPath` to redirect); “Packages that only provide skills, prompt templates, or themes do not show a changelog.”

Dependency rules: “Put runtime packages imported by extensions in `dependencies`” — “`devDependencies` are not installed.” Host-provided packages (`@earendil-works/pi-ai`, `@earendil-works/pi-agent-core`, `@earendil-works/pi-coding-agent`, `@earendil-works/pi-tui`, `typebox`) go in `peerDependencies` with `"*"`; “Do not list host-provided packages in `dependencies`. A physical copy can bypass Pi's extension module mapping in compiled ESM and create duplicate classes, registries, and initialization work.” “Installed packages load with separate module roots.”

## Official sources

- [Extensions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md) · [Skills](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md) · [Prompt Templates](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/prompt-templates.md) · [Pi Packages](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/packages.md) · [Themes](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/themes.md) · [TUI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/tui.md)
