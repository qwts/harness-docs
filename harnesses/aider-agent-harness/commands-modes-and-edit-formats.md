# Commands, modes, and edit formats

**Verified:** 2026-11-15

## Four built-in chat modes

| Mode | Behavior |
|---|---|
| `code` | Aider makes changes to your code to satisfy requests (the default) |
| `ask` | Discussion and questions only; never makes changes |
| `architect` | Changes files, but an architect model proposes and a separate editor model applies the edits |
| `help` | Questions about aider itself: usage, configuration, troubleshooting |

Switching is per message (`/code`, `/ask`, `/architect`, `/help` prefixes), sticky for the session (`/chat-mode <mode>`), or at launch (`--chat-mode <mode>`, with `--architect` as the documented shortcut). The prompt shows the active mode (`>`, `ask>`, `architect>`). The documented workflow is to plan in `ask` mode and execute tersely in `code` mode.

Architect mode costs two LLM requests: the main model proposes changes as plain text, then the editor model (`--editor-model`, `--editor-edit-format`) translates the proposal into edits using an editor edit format. `--auto-accept-architect` (default True) controls whether the proposal is applied without confirmation.

## Edit formats

Aider asks the model to return edits in a structured format; the optimal format is configured per model, and `--edit-format` forces one:

| Format | Mechanism |
|---|---|
| `whole` | Returns a complete updated copy of each changed file; simple but slow and costly |
| `diff` | Search/replace blocks (syntax resembling git conflict markers); returns only changed parts |
| `diff-fenced` | Like `diff` but the file path sits inside the code fence; used mainly with Gemini models |
| `udiff` | Simplified unified diffs; used mainly with the GPT-4 Turbo family to counter lazy coding |
| `editor-diff` / `editor-whole` | Streamlined `diff`/`whole` variants for the editor model in architect mode |

## In-chat commands

The commands reference lists more than forty slash commands. The load-bearing ones:

| Group | Commands |
|---|---|
| Context | `/add`, `/drop`, `/ls`, `/read-only`, `/map`, `/map-refresh`, `/tokens`, `/clear`, `/reset`, `/paste`, `/copy-context`, `/context` (context mode for surrounding code) |
| Editing flow | `/code`, `/ask`, `/architect`, `/help`, `/chat-mode`, `/ok` (added on `main` after v0.86.0; asks aider to proceed with the proposed changes) |
| Git | `/diff`, `/undo`, `/commit`, `/git` |
| Shell and repair | `/run` (alias `!`) runs a shell command and optionally shares output; `/test` runs the test command and shares output on non-zero exit; `/lint` lints and fixes in-chat or dirty files |
| Models | `/model`, `/models`, `/editor-model`, `/weak-model`, `/reasoning-effort`, `/think-tokens` |
| Session | `/save` (write a command file that reconstructs the chat’s files), `/load`, `/settings`, `/editor` or `/edit`, `/multiline-mode`, `/voice`, `/web`, `/copy`, `/report`, `/exit`, `/quit` |

`/save` and `/load` (plus `--load` at launch) provide command files — aider’s closest documented equivalent to saved workflows. There is no documented command for MCP servers, subagents, skills, or hooks.

## Watch files: IDEs without a plugin

`--watch-files` (default False) watches repo files and looks for one-liner comments that start or end with `AI`, `AI!`, or `AI?` in `#`, `//`, or `--` comment styles. A plain `AI` comment is noted but inert; `AI!` triggers aider to make the requested changes; `AI?` triggers an answer to the question. Several `AI` comments can be scattered across files and collected by one final `AI!`. This is the documented way to use aider from any IDE or editor: save files with AI comments, and aider responds in its terminal session. Aider asks the model to remove the AI comments from the code as part of the change.

## Web, voice, editor, and notifications

- `/web` scrapes a URL to markdown and sends it in a message; scraping uses Playwright, which aider prompts to install on first use unless `--disable-playwright` is set.
- `/voice` records and transcribes voice input; `wav` is the default `--voice-format` (webm/mp3 need ffmpeg), with `--voice-language` and `--voice-input-device` available.
- `/editor` opens a text editor to compose a prompt. The editor comes from `--editor`/YAML, then `AIDER_EDITOR`, `VISUAL`, `EDITOR`; defaults are `notepad` (Windows), `vim` (macOS), `vi` (Linux/Unix). The editor must run in blocking mode (e.g. `code --wait`).
- `--notifications` (default False) rings the terminal bell when a response is ready; `--notifications-command` runs a custom command instead.
- `--multiline` enables Meta-Enter submission; `/multiline-mode` toggles it by swapping Enter and Meta-Enter. `--vim` selects vi editing mode.

## Official sources

- [Chat modes](https://aider.chat/docs/usage/modes.html)
- [Edit formats](https://aider.chat/docs/more/edit-formats.html)
- [In-chat commands](https://aider.chat/docs/usage/commands.html)
- [Watch files](https://aider.chat/docs/usage/watch.html)
- [Images & web pages](https://aider.chat/docs/usage/images-urls.html)
- [Voice-to-code](https://aider.chat/docs/usage/voice.html)
- [Editor configuration](https://aider.chat/docs/config/editor.html)
- [Notifications](https://aider.chat/docs/usage/notifications.html)
- [Prompt caching](https://aider.chat/docs/usage/caching.html)
