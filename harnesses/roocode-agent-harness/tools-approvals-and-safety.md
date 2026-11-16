# Tools, approvals, and safety boundaries

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site main — product sunset May 15, 2026)

## Tool system

“Roo Code implements a sophisticated tool system that allows AI models to interact with your development environment in a controlled and secure manner.” Tools are grouped: **Read** (`read_file`, `list_files`, `read_command_output`), **Search** (`search_files`, `codebase_search`), **Edit** (`apply_diff`, `apply_patch`, `edit`, `edit_file`, `search_replace`, `write_to_file`), **Image** (`generate_image`), **Command** (`execute_command`, `run_slash_command`*), **MCP** (`use_mcp_tool`, `access_mcp_resource`), **Workflow** (`switch_mode`, `new_task`, `ask_followup_question`, `attempt_completion`, `update_todo_list`, `skill`).

“Certain tools are accessible regardless of the current mode”: `ask_followup_question`, `attempt_completion`, `switch_mode`, `new_task`. The workflow is approve-first: “Present the tool with its parameters for your review,” then “Execute the approved tool and show you the results” — unless auto-approval or a queued message overrides it.

## Auto-approval

The docs lead with a warning: “Auto-approve settings bypass confirmation prompts, giving Roo direct access to your system. This can result in **data loss, file corruption, or worse**. Command line access is particularly dangerous.” Controls: an Auto-Approve dropdown with “Enabled (bottom-right): master pause/resume for auto-approval”, “All / None” bulk chips, and per-permission tiles — “you can keep Enabled On with "None” selected.” Permission tiles: “Read files and directories” (Medium), “Edit files” (**High**), “Execute approved commands” (“Runs whitelisted terminal commands automatically”, **High**), “Use the browser” (Medium), “Use MCP servers” (Medium-High), “Switch modes” (Low), “Create & complete subtasks” (Low), with more controls in Settings. Toggle: `Cmd+Alt+A` / `Ctrl+Alt+A` or Command Palette `roo-cline.toggleAutoApprove`.

The queue caveat bears repeating: “When a queued message is processed, Roo implicitly approves whatever would normally require your confirmation …—even if auto-approval is disabled.”

## Checkpoints

“Checkpoints automatically version your workspace files during Roo Code tasks, enabling non-destructive exploration of AI suggestions.” “**Checkpoints are enabled by default**” and “Git must be installed for checkpoints to function”; “The shadow Git repository operates independently from your project's existing Git configuration.” Timing: “Checkpoints are recorded when tasks begin and before file modifications. They are not automatically created before command execution.” A “Checkpoint initialization timeout” (10–60 seconds, default 30s) guards slow initializations.

## `.rooignore`

“.rooignore … allows you to specify files and directories that Roo should not access or modify, similar to how `.gitignore` works.” “The syntax for `.rooignore` is identical to `.gitignore`.” Enforcement spans tools and mentions; strict tools “directly check `.rooignore` before any file operation. If a file is ignored, the operation is blocked.” “Roo actively monitors the `.rooignore` file. Any changes you make are reloaded automatically, ensuring Roo always uses the most current rules.” Plus: “The `.rooignore` file itself is always implicitly ignored, so Roo cannot change its own access rules.”

## Terminal, diagnostics, and context management

- **Shell integration** — “Terminal Shell Integration is a key feature that enables Roo Code to execute commands in your terminal and intelligently process their output,” with documented troubleshooting for shells that don't emit VS Code's OSC markers.
- **Diagnostics integration** — VS Code Problems/LSP diagnostics feed task context.
- **Concurrent reads/edits** — “Concurrent File Reads” is a documented feature; “Concurrent File Edits” and “Background Editing” sit under experimental features.
- **Intelligent Context Condensing** — “helps manage long conversations by summarizing earlier parts of the dialogue … This feature is **enabled by default**.”
- **Codebase indexing** — “creating a semantic search index using AI embeddings”: “**Parses your code** using Tree-sitter … **Creates embeddings** … **Stores vectors** in a Qdrant database … **Provides the `codebase_search` tool**.” (The FAQ: it “creates a semantic search index of your project using AI embeddings.”) External dependencies (Qdrant + an embeddings provider) are required; a free path via “Qdrant Cloud (free tier) or Docker Qdrant” + Gemini is documented.

## Experimental features

“Roo Code includes experimental features that are still under development. These features may be unstable, change significantly, or be removed in future versions.” Enabled under the “Advanced Settings” → “Experimental Features” settings sections: **Custom Tools** (“Define TypeScript or JavaScript tools that Roo can call like built-in tools”, in `.roo/tools/` (project) / `~/.roo/tools/` (global), Zod-validated — “Custom tools are **automatically approved** when enabled”), **Background Editing**, **Image Generation**, and **Run Slash Command**.

## Official sources

- [Tool Use Overview](https://roocodeinc.github.io/Roo-Code/advanced-usage/available-tools/tool-use-overview) · [How Tools Work](https://roocodeinc.github.io/Roo-Code/basic-usage/how-tools-work) · [Auto-Approving Actions](https://roocodeinc.github.io/Roo-Code/features/auto-approving-actions) · [Checkpoints](https://roocodeinc.github.io/Roo-Code/features/checkpoints) · [.rooignore](https://roocodeinc.github.io/Roo-Code/features/rooignore) · [Shell Integration](https://roocodeinc.github.io/Roo-Code/features/shell-integration) · [Context Condensing](https://roocodeinc.github.io/Roo-Code/features/intelligent-context-condensing) · [Experimental Features](https://roocodeinc.github.io/Roo-Code/features/experimental/experimental-features) · [Custom Tools](https://roocodeinc.github.io/Roo-Code/features/experimental/custom-tools)
