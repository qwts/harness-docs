# Workspaces and sessions

**Verified:** 2026-11-15

## Session model

“Run `mirror` in a project directory to start a Mirror session. The first time you start Mirror in a directory, it asks you to confirm that you trust that directory's files.” One session holds four things, per the overview: a **workspace** (“A directory with its own agent and its own conversation”), a **conversation** (“The history of prompts, agent replies, and tool calls in a workspace. Mirror saves every conversation in `~/.mirror`, so you can resume one later or branch from an earlier turn”), **tools** (“Tools run on your machine with your account's permissions”), and **instructions and skills**. “Workspaces are independent: switching to another one doesn't pause the agent you left, and each keeps its own model, reasoning effort, and approval mode.”

The screen shows, top to bottom: the transcript (with `detail`/`default`/`focus` view modes via `Ctrl+T` or `/view`), the prompt, the settings line (“The approval mode (`manual` or `auto`), the model, and the reasoning effort on the left, and generation speed and context use on the right”), the repository line (“The current branch and a count of uncommitted changes”), and workspace tabs once there is more than one workspace.

## Worktree workspaces

“A workspace is a directory with its own agent and conversation. Mirror starts with one, for the directory you launched it in.” `/worktree <name>` “fetches the remote's default branch and creates a new checkout of it on a branch called `mirror/<name>`. Mirror keeps the checkout under `~/.mirror/worktrees/`.” It “uses `origin` when present, or the sole remote otherwise”; “If fetching fails, it uses the cached default branch or local committed revision and shows a warning. If there are several remotes and none is named `origin`, configure `origin` or create a checkout yourself and open it with `/folder`.”

What is and isn't carried over:

- “Local edits and untracked files aren't copied. A checkout based on the remote's default branch also excludes your unpushed commits. If Mirror falls back to the local committed revision, those commits may be included.”
- “The new workspace starts a fresh conversation and copies the model, reasoning effort, and approval mode of the workspace you created it from. If you launched Mirror in a subdirectory, the new agent starts in the same subdirectory of the new checkout.”
- Naming: “A name has up to 64 letters, digits, `.`, `-`, and `_`, starts with a letter or digit, and can't contain `..` or end in `.` or `.lock`. Mirror creates one workspace at a time.” Omit the name and “Mirror picks one.”
- `/fork` comparison: `/worktree` gets “A checkout of the remote's default branch, with a cached or local committed fallback. Local edits aren't copied” and “Starts fresh”; `/fork` gets “The current committed revision plus working changes and non-ignored untracked files” and “Continues from a copy of the current history.”

## Fork, folder, and sandbox workspaces

“Use `/fork [name]` when you want to try another approach with the conversation and files you already have”: “Mirror captures the current working files, creates a new Git worktree or Jujutsu workspace, and branches the conversation into it.” Constraints: “Forking requires an idle local workspace in the repository where you launched Mirror. Wait for the turn and queued messages to finish first. Hosted and sandbox workspaces can't be forked.” “Ignored files, submodule contents, and nested repositories aren't copied. In Git, copied working changes are unstaged. Resolve Jujutsu conflicts before forking. Stop other processes that are writing to the checkout while Mirror captures it, since the capture isn't atomic.” A fork “captures the files as they are now; it doesn't restore files from an earlier message. If creation fails, inspect the conversation ID and path in the error before retrying, since a partial checkout may remain.”

`/folder <path>` opens “an existing directory” as a workspace. `/sandbox [name]` creates a sandbox workspace: “Run `/sandbox [name]` to choose an empty workspace or a copy of the current repository's committed revision. Mirror stores the clone under `~/.mirror/sandboxes/<name>`. Its file and shell tools run in a microVM; extensions and local MCP servers still run on the host. Setup asks before installing the runtime or downloading an image.” Platform bounds: “Sandbox workspaces require macOS on Apple Silicon, or Linux with access to `/dev/kvm`. Quitting Mirror stops the VMs; reopening it restarts them. `/drop` deletes the sandbox's VM and clone after confirmation.”

## Switching, status, and identity

Switch with `Tab`/`Shift+Tab`, `Ctrl+1` through `Ctrl+9` (“in terminals that support these keys”), or `Ctrl+O` / `/workspaces` for “a searchable list of workspaces” (type to search by name or directory; the current workspace is marked `*`). `Ctrl+N` opens “Choose a new workspace in the current directory or a new worktree.” “Switching never stops an agent. Each one keeps its conversation, and any turn in progress keeps running while you work elsewhere.”

Agent status indicators: `○` “Idle, waiting for a task”, `●` “Working”, `!` “Waiting for you to approve a tool call”, `✓` “Finished while you were in another workspace. Clears when you switch to it”, `×` “The agent failed”, `·` “Not started yet”, `↻` “A saved conversation, which resumes when you select it.” “With manual approval, an agent that needs approval waits at `!` until you switch to it and answer.”

`/rename` changes “the name shown in the tabs and the workspace list” only: “The branch, directory, and conversation keep their names.” `/tag` or `Ctrl+S` picks a workspace icon. Each workspace “remembers its own model, reasoning effort, and approval mode … Mirror stores these settings in `~/.mirror/cli.db`.” Setting `"display_unmanaged_worktrees": true` in `config.json` makes Mirror also show “Git worktrees or Jujutsu workspaces you created yourself”; those are closed with `/close`, since “`/drop` can't delete folder workspaces.”

## Review, merge, and teardown

Each workspace “is an ordinary checkout on its own branch, so you review and merge its work with your usual tools. The line above the tabs shows the workspace's branch and how many tracked files have uncommitted changes.” Commit on the `mirror/<name>` branch, then merge it or open a pull request from it.

`/close` closes “a workspace while keeping its files.” `/drop` removes it: “Mirror removes the checkout, closes its agent, and switches you to another workspace.” The warning is blunt: “`/drop` deletes the checkout, including uncommitted changes, without asking first. Commit or copy anything you want to keep before you run it.” “The `mirror/<name>` branch and the workspace's saved conversations remain. To reuse the name for a new workspace, delete the old branch first.” “You can't drop the repository's main checkout (the one you cloned) or your only workspace. `/drop` doesn't run while the agent is working; wait or press `Escape` first.”

## Conversations, resume, and rewind

| Action | Behavior |
|---|---|
| `/clear` | “Start a new conversation in the current workspace” (idle only) |
| `/resume [id]` | “Choose a saved conversation, or resume one by its ID or a unique prefix of it”; `Tab` in the list widens “from this directory to the repository to all directories” |
| `mirror --continue` | “Resume each workspace's most recent conversation”; otherwise each workspace starts a new conversation when you reopen it |
| `--conversation-id <id>` | “Resume one conversation. Can't be combined with `--continue`” |
| `/rewind` | “Edit an earlier prompt and continue from there as a new branch” — “Files aren't restored” |
| `/fork [name]` | Copies the conversation into a new workspace |

Rewind carries its own warning: “Rewinding changes only the conversation. Files the agent edited after that point stay as they are. Use version control to undo file changes.” The CLI reference notes the original branch is preserved: “the original keeps its later turns.”

## Long conversations

“When a conversation approaches the model's context window, Mirror summarizes older history so the agent can keep working.” Launch options: `--context-window <tokens>` (“Override the context length Mirror assumes. Use it for a local model whose limit Mirror can't detect”), `--compaction-threshold <tokens>` (“Summarize history once a prompt reaches this many tokens, instead of at 90% of the context window”), and `--compaction-model <id>` (“Use a different model to write the summaries”). “Run `/compact` while idle to compact sooner. Press `Escape` to cancel compaction; a cancelled or failed summary doesn't replace the context.”

## Jujutsu and quitting

“In a Jujutsu repository, `/worktree <name>` fetches the selected remote's default branch and adds a Jujutsu workspace called `<name>` at that revision. … The `jj` command must be installed. The repository line shows the change ID and any bookmarks.”

The automation boundary is absolute: “Quitting Mirror stops every agent. Nothing keeps running in the background after you exit.”

## Official sources

- [Mirror CLI overview](https://developers.reflection.ai/mirror-overview.md) · [Workspaces](https://developers.reflection.ai/mirror-workspaces.md) · [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md)
