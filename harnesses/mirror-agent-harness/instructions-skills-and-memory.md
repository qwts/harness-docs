# Instructions, skills, and memory

**Verified:** 2026-11-15

## Instructions (AGENTS.md)

“Put project guidance in an `AGENTS.md` file. Mirror reads it before the agent starts, so it's the place for anything you'd otherwise repeat in every prompt.” Read order:

1. “`~/.mirror/AGENTS.md`, for guidance that applies to all your projects.”
2. “Every directory from the repository root down to the directory where the workspace runs. A file in a subdirectory adds to the root's file.”

“Outside a repository, Mirror reads the global file and the current directory only.” Compatibility: “Mirror also reads `CLAUDE.md`, so a project that already has one works without changes. If a directory has both, Mirror uses `AGENTS.md`.” Cap: “Keep instructions short and specific. Mirror reads at most 32 KiB of instructions in total and truncates the rest. To see which files were read and how much of the context window they use, run `/context`.”

There is no hook or plugin layer between instructions and behavior — project-side customization points are `AGENTS.md`, skills (`.agents/skills/`, `.claude/skills/`), and extensions (`.mirror/extensions/`); a dedicated hooks system does not exist in the documentation.

## Skills

“A skill is a directory containing a `SKILL.md` file. The file starts with a name and a description, followed by instructions for the task.” Frontmatter fields: `description` (required — “Mirror skips a skill that has no description”), `name` (optional, “Defaults to the directory name. It must start with a letter or digit and contain only letters, digits, `-`, and `_`”), and `disable-model-invocation` (“Set to `true` to keep the agent from choosing the skill on its own, so it runs only when you mention it”).

Locations: `~/.mirror/skills/<skill>/SKILL.md` is global (“In every project”); project skills live in `.agents/skills/<skill>/SKILL.md` or `.claude/skills/<skill>/SKILL.md` “in directories from the repository root down to the working directory.” “Mirror also finds skills in subdirectories of these locations, up to six levels deep. If two skills have the same name, the one closer to the working directory wins.”

Invocation: “The agent picks a skill on its own when your request matches its description, so a clear description matters most. To ask for one explicitly, type `$` to browse skills, or mention one as `$<name>` anywhere in your prompt” (for example `$review-tests Check the parser changes on this branch.`). Cost model: “Unlike instructions, skills cost almost nothing until they're used, so you can keep many of them.”

## Memory

“Mirror can keep a private memory of how you work, including your preferences, facts about your repositories, and approaches that worked or failed. Memory is off until you turn it on, and it stays on your machine.”

- Enable: “Run `/settings`, choose **Memory**, and use the on/off switch. The change applies to this session and future local sessions. Finish or interrupt an active turn before changing it. The Memory setting is unavailable in remote sessions or when the memory extension is disabled.”
- Status and import: per the command table, `/memory` or `/memory status` shows whether memory is on, and `/memory import` will “Import the memories that Claude Code and Codex keep on this machine.”
- Storage: “Mirror stores memories in `~/.mirror/memory`. Memories the agent should always see are kept as bullets in `~/.mirror/PINNED_MEMORIES.md`, which you can also edit by hand.”

## Official sources

- [Mirror instructions and skills](https://developers.reflection.ai/mirror-instructions-and-skills.md) · [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md)
