# Configuration, project context, memory, and skills

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; app v3.18.2)

## Project instructions (AGENTS.md)

“To keep long-term instructions for ZCode Agent, add an AGENTS.md instructions file. ZCode reads it when starting a task.” Exactly two sources:

- “User global instructions” — `~/.zcode/AGENTS.md` (“Good for cross-project personal preferences”)
- “Workspace instructions” — AGENTS.md in the current Workspace

“When both sources exist, ZCode appends the user global instructions first, then the workspace instructions. The workspace instructions are treated as the primary project source for the current task. When neither source exists, no project instructions are injected.” Hard limits: “ZCode only reads the user global AGENTS.md and the current Workspace AGENTS.md. It does not merge multiple AGENTS.md files across directory levels. It also does not scan child directories, expand @import / @include, or choose rule files automatically by task type.” CLAUDE.md: “CLAUDE.md is not continuously read by ZCode Agent at runtime … ZCode only uses it during onboarding as a one-time migration source: existing content can be copied into AGENTS.md, and future runs use AGENTS.md.”

## Context entries

Input-box triggers: “Add Attachment” (uploads), “@” — “Reference a file or an entire folder in the workspace,” “#” — “Link a past conversation to bring its context into the current task,” “/” — “Call a saved command,” and “$ to call skills.” “Pasting a large block of text won't flood the composer — anything past a certain length automatically becomes an attachment.” Selected text from any reply can be added “to current task”; “Only single-line selections work right now,” with caps — going over “produces a message rather than a silent truncation”.

## Project memory

“AGENTS.md is something you maintain by hand; Memory is what the Agent accumulates on its own. Once enabled, ZCode distills reusable facts in the background after each successful turn and brings them along automatically when you start a new session in that project.” It remembers four kinds of information: **User** (“your identity, expertise, and preferences”), **Feedback** (“Corrections or confirmations about how the Agent should work”), **Project** (“Project goals, constraints, and progress”), and **Reference** (“External resources you've shared”).

Rules: “Memory is off by default and requires ZCode v3.6.4 or later”; “Turn it on under Settings → General → Memory”; “It applies to new sessions … conversations already in progress keep whatever setting they started with”; “It can increase token usage.” Manage in conversation — “say something like “remember that we deploy from the staging branch” … or “forget what I said about the deploy branch” to remove it”. Limits: “You can't browse or clear memories yet,” and “Project memory applies to the main conversation only — subagents neither read nor write it.” The v3.18.2 changelog adds “new toggles for project editing, per-device memory, and mobile push notifications.”

## Skills

“A Skill is a reusable working instruction. It is usually defined by a SKILL.md file that describes when the skill should be used, how the Agent should work, and what kind of output is expected.”

- **Location** — user-level skills live at “`~/.zcode/skills/<skill-name>/SKILL.md`”; “The directory name is the skill name, and that is the name you reference in chat.” Workspace-scope skills are also surfaced in Settings → Skills by source tag.
- **Format limits** — “The frontmatter must include `name` and `description`; a skill missing either is ignored, with the reason shown in the settings-page diagnostics”; “`description` is capped at 1024 characters. Going over drops the whole skill”; “A SKILL.md body over 100KB is truncated when loaded.”
- **Context mechanics** — “Each turn injects the metadata of every enabled skill (name plus a description excerpt of up to 250 characters) into the model context; bodies are loaded on demand only when a skill is invoked.” “All skills share one fixed metadata budget: install enough skills to blow past it and the injection degrades to names only, leaving the model unable to tell when to use which skill — automatic trigger rates drop sharply.” “Disabled skills are neither injected nor invocable,” and “The / panel and the model see the same skill list — there is no switch that shows a skill to the panel but hides it from the model.”
- **Distribution** — “ZCode has no standalone skill marke[t]”: skills ship via plugins or are created in-app (“Click New Skill … ZCode will guide you through generating a new skill with the Agent in chat”). Subagent tool allowlists can lock skills out (“the skill tool may not be on it, and the subagent then cannot invoke any skill”).

## Commands

“Commands invoke built-in ZCode Agent capabilities and can also save prompts you use often” — invoked from the input box with `/`. Built-ins: `/goal` — “Show, set, replace, pause, resume, or clear the current session goal” and `/compact` — “Compact the current conversation context while preserving key information”. Custom commands: Scope “User (available in all workspaces) or Workspace (current project only)”; “Custom commands are stored as `.md` files under `~/.zcode/commands` (workspace-level commands live in the project directory).” Fields: Name, Description, “Argument hint … such as `<file-path>`”, Prompt. “Import commands from external Agent” brings Claude Code commands into ZCode.

## Subagents

“A subagent is a specialized agent that the primary Agent can launch to handle work in its own isolated context, then summarize the results back into the main conversation.” Built-ins: **general-purpose** (“access to all tools”) and **Explore** (“read-only … It does not create, modify, move, or delete files”). Custom subagents are **Beta**: “You can now create your own subagents from Settings -> Subagents” with “its own model, tool permissions, and instructions” — fields: Name, Color (“an identity marker only, not a status”), Model (“Inherit default”), Thinking effort (“Only takes effect when a specific model is set”), Description (what the primary Agent uses to pick it), Available tools (“All permissions by default” or “Custom tools” lets you check them one by one; writable tools such as Bash / Edit / Write are flagged), System prompt. “When you save, ZCode writes the subagent as a Markdown file under `~/.zcode/agents/<name>.md`, and the ZCode Agent runtime loads it on the next run”; reference with `@`. Frontmatter keys are “camelCase, case-sensitive”; “name / description” — “Required. A file missing either is ignored, with a diagnostic”. “Built-in roles cannot be edited or deleted, but you can assign them a dedicated model and thinking effort.”

## Task & file management

“Task views let you organize the task list by group, workspace, or timeline”; archived tasks use “the retention window (configurable as 3 / 7 / 14 / 30 days)” for auto-archive candidates. Task groups offer “7 colors (gray / red / orange / yellow / green / blue / purple).” The **Repo wiki** turns the repository into “an architecture guide to the codebase, not a product documentation site or a user manual … Every claim carries a source location you can click through to,” stored “in the runtime environment the current workspace lives in, one per workspace,” sending code context “filtered by its safety rules … to the model service you selected.” **Edit History** revises sent messages: “only the user message of the last turn can be edited, and editing is unavailable while the task is running or has queued messages.” The **Command Center** (Command/⌘ + K) covers “commands, conversations, and files at once.”

## Official sources

- [ZCode Agent](https://zcode.z.ai/en/docs/agents) · [Memory](https://zcode.z.ai/en/docs/memory) · [Skill](https://zcode.z.ai/en/docs/skill) · [Command](https://zcode.z.ai/en/docs/commands) · [Subagents](https://zcode.z.ai/en/docs/subagents) · [Task & File Management](https://zcode.z.ai/en/docs/task-management) · [Wiki](https://zcode.z.ai/en/docs/repo-wiki) · [Edit History](https://zcode.z.ai/en/docs/edit-history)
