# Instructions, skills, and commands

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site main — product sunset May 15, 2026)

## Custom instructions and rules

“Custom Instructions allow you to personalize how Roo behaves.” Layers and locations:

- **Global rules** — “Linux/macOS: `~/.roo/rules/` and `~/.roo/rules-{modeSlug}/`” (Windows `%USERPROFILE%\.roo\...`).
- **Workspace rules** — “Preferred Method: Directory (`.roo/rules/`)” with ordered files (“01-general.md”, …); “Fallback Method: Single File (`.roorules`)”.
- **Mode-specific rules** — “Preferred Method: Directory (`.roo/rules-{modeSlug}/`)”; “Fallback Method: Single File (`.roorules-{modeSlug}`)”.
- “Rules are loaded in order: Global rules first, then workspace rules. If there's a conflict, workspace rules take precedence.”

Global custom instructions can also be entered in the Prompts tab under “Custom Instructions for All Modes.”

## Skills

“Skills package task-specific instructions that Roo loads on-demand when your request matches the skill's purpose.” Locations: project `.roo/skills/` (“Version-control project skills in `.roo/skills/` for consistent team workflows”) and global `~/.roo/skills/`; “Project skills override global skills, mode-specific override generic.”

“Skills use **progressive disclosure**”: “**Level 1: Discovery** - Roo reads each `SKILL.md` file and parses its frontmatter to extract `name` and `description`. Only this metadata is stored for matching—the full content isn't held in memory until needed. **Level 2: Instructions** - When your request matches a skill's description, Roo uses `read_file` to load the full SKILL.md instructions into context. **Level 3: Resources** - The prompt tells Roo it may access bundled files (scripts, templates, references) alongside the skill. There's no separate resource manifest—Roo discovers these files on-demand.” Skills can be mode-targeted (“skills that only activate in specific modes”).

## Slash commands

“Create custom slash commands to automate repetitive tasks … with simple markdown files.” Type `/` in chat to select one; creation via Settings → Slash Commands or files in `.roo/commands/` (project) / `~/.roo/commands/` (global). The `run_slash_command` tool executes them from the model side and is marked “*(Experimental - requires enablement)*” / “Experimental feature - requires explicit enablement in settings”.

## Marketplace

“The Roo Code Marketplace is a central hub for discovering and installing community-contributed extensions, known as MCPs (Model Context Protocol) and Modes.” “Install and remove items with a single click,” with “Project & Global Scopes: Install items for a specific project or for all your projects.”

## Prompt structure

Three primary message types go to the LLM — “System Prompt”, “User Messages”, “Assistant Messages” — plus at the API level “Tool Messages: Results returned from tool executions.” The system prompt is generated per interaction and contains “Role Definition … Tool Descriptions … Tool Use Guidelines … Capabilities … Available Modes … Operational Rules … System Information … Custom Instructions.” User messages append “Environment Details: Automatically appended information about your workspace state” — open files/tabs, cursor position, active terminals with output, recently modified files.

## Official sources

- [Custom Instructions](https://roocodeinc.github.io/Roo-Code/features/custom-instructions) · [Skills](https://roocodeinc.github.io/Roo-Code/features/skills) · [Slash Commands](https://roocodeinc.github.io/Roo-Code/features/slash-commands) · [Marketplace](https://roocodeinc.github.io/Roo-Code/features/marketplace) · [Prompt Structure](https://roocodeinc.github.io/Roo-Code/advanced-usage/prompt-structure) · [Context Mentions](https://roocodeinc.github.io/Roo-Code/basic-usage/context-mentions)
