# Modes, orchestration, and task workflow

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site main — product sunset May 15, 2026)

## The five built-in modes

“Modes in Roo Code are specialized personas that tailor the assistant's behavior to your current task. Each mode offers different capabilities, expertise, and access levels.” Switching: dropdown, slash commands (“Type `/architect`, `/ask`, `/debug`, `/code`, or `/orchestrator` at the beginning of your message”), keyboard cycling (macOS ⌘ + ., Windows/Linux Ctrl + . — “Each press cycles through the available modes in sequence”), and accepting Roo's suggestions.

| Mode | Tool access |
|---|---|
| `💻 Code` (default) | “Full access to all tool groups: `read`, `edit`, `command`, `mcp`” |
| `❓ Ask` | “Limited access: `read`, `mcp` only (cannot edit files or run commands)” |
| `🏗️ Architect` | “`read`, `mcp`, and restricted `edit` (markdown files only)” |
| `🪲 Debug` | Full groups, with “custom instructions to reflect, distill possibilities, add logs, and confirm before fixing” |
| `🪃 Orchestrator` (aka Boomerang Mode) | “No direct tool access (uses `new_task` tool to delegate work to other modes)” |

Tool groups: `read` — “File reading, listing, and searching capabilities”; `edit` — “File modification and creation capabilities”; `command` — “Terminal command execution”; `mcp` — “Model Context Protocol server interactions.”

Persistence: “your selected mode persists between sessions—Roo remembers which mode you were using when you return,” and Sticky Models — “Each mode remembers your last-used model. When switching modes, Roo automatically selects that model.”

## Custom modes

“Custom modes can be either **global** (available across all projects) or **project-specific** (defined within a single project).” Defining properties: `slug`, `name`, `description`, `roleDefinition` (“placed at the beginning of the system prompt”), `groups` (toolsets and file permissions; edit-group file restrictions via YAML/JSON), `whenToUse` (“guidance for Roo's automated decision-making, particularly for mode selection and task orchestration … used by … the `🪃 Orchestrator` mode”), and `customInstructions` (“added near the end of the system prompt”). “With the introduction of the `description` field, the `roleDefinition` should provide a detailed description of the mode's identity.” Import/Export packages “any mode—and its associated rules—into a single, portable YAML file.” The Roo Code Marketplace distributes community modes (“browse and install community-contributed modes with a single click”).

## Boomerang tasks (orchestration)

“Boomerang Tasks (also known as subtasks or task orchestration) allow you to break down complex projects into smaller, manageable pieces using the built-in **`🪃 Orchestrator` Mode (aka Boomerang Mode)**.” Flow: “Roo analyzes a complex task and suggests breaking it down into a subtask”; “The parent task (in Orchestrator mode) pauses, and the new subtask begins in a different, specialized mode”; “When the subtask's goal is achieved, Roo signals completion”; “The parent task resumes with only the summary” of the subtask. “Each subtask operates in its own isolated context with a separate conversation history.” The delegation primitive is the `new_task` tool; `switch_mode` changes modes mid-task.

## In-chat workflow features

- **Message queueing** — “send multiple messages while Roo is working, and they'll be processed sequentially.” Security-relevant: “When a queued message is processed, Roo implicitly approves whatever would normally require your confirmation (tool calls, file writes, running commands)—even if auto-approval is disabled.”
- **Suggested responses** — when Roo calls `ask_followup_question`, “Roo often provides suggested answers alongside the question” as clickable buttons.
- **Task todo list** — `update_todo_list` tracks multi-step work in the task view.
- **Checkpoints** — automatic shadow-Git snapshots around file edits (see the tools and approvals page).
- **Worktrees** — “Git worktrees allow you to work on multiple branches of the same repository simultaneously by keeping them in different directories. Each worktree gets its own VS Code window with Roo Code.” Requirements: “Multi-root workspaces are not supported” and “Workspace must be at the repository root (not a subfolder).”
- **Context mentions** — `@/path/to/file.ts` (file contents), `@/path/to/folder/` (non-recursive), `@problems` (VS Code Problems panel), `@terminal`, `@a1b2c3d` (commit), `@git-changes`, `@https://example.com` (URL import), and `/<command-name>` for slash commands. “`.rooignore` … affects both Roo's tools and context mentions.”
- **Enhance prompt / code actions / image generation / text-to-speech** — auxiliary features documented under features; image generation is also listed among experimental features.

## Official sources

- [Using Modes](https://roocodeinc.github.io/Roo-Code/basic-usage/using-modes) · [Customizing Modes](https://roocodeinc.github.io/Roo-Code/features/custom-modes) · [Boomerang Tasks](https://roocodeinc.github.io/Roo-Code/features/boomerang-tasks) · [Worktrees](https://roocodeinc.github.io/Roo-Code/features/worktrees) · [Message Queueing](https://roocodeinc.github.io/Roo-Code/features/message-queueing) · [Context Mentions](https://roocodeinc.github.io/Roo-Code/basic-usage/context-mentions) · [Task Todo List](https://roocodeinc.github.io/Roo-Code/features/task-todo-list)
