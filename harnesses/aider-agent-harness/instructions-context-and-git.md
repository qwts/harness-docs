# Instructions, context, and git

**Verified:** 2026-11-15

## The chat is explicit, not ambient

Aider documents no nested memory-file hierarchy. Context is what you put in the chat: `--file` / `/add` add editable files, `--read` / `/read-only` add reference-only files, and the repository map summarizes the rest. Aider always reads the latest copy of files from the filesystem when you send each message, so external edits while aider runs are seen on the next message.

Read-only files are part of the chat but not the edit surface. Files listed in `.gitignore` are not added to aider’s editing scope unless `--add-gitignore-files` is enabled.

## Conventions load only on request

The documented way to give the model project conventions is a small markdown file loaded into the chat — the docs use `CONVENTIONS.md` with entries like “Prefer httpx over requests”. It is not discovered automatically. The recommended loading is `aider --read CONVENTIONS.md` at launch or the in-chat `/read-only CONVENTIONS.md` command, which marks the file read-only and makes it cacheable when prompt caching is enabled. (The conventions guide still spells the in-chat command `/read`; the current CLI implements only `/read-only`, so that spelling is outdated.) To load conventions every session, add a `read:` entry to `.aider.conf.yml`. A community conventions repository exists for sharing files.

## The repository map is a budgeted summary

Aider sends a concise map of the whole git repository with each change request: the most important classes, functions, types, and call signatures, chosen by a graph ranking algorithm over file dependencies, fitted into an active token budget.

- `--map-tokens` defaults to 1k tokens; `0` disables the map.
- The map expands significantly when no files have been added to the chat (`--map-multiplier-no-files`, default 2).
- `--map-refresh` controls refresh timing: `auto` (default), `always`, `files`, or `manual`; `/map` prints it and `/map-refresh` forces a refresh.
- Map construction relies on tree-sitter; parsers come from the tree-sitter-language-pack, and repo-map support for a language additionally needs that language’s `tags.scm` file.
- `.aiderignore` (gitignore-style syntax, `.aiderignore` in the git root by default, configurable with `--aiderignore`) removes paths from consideration; `--subtree-only` restricts aider to the current subtree for very large repos.
- The docs warn that aider is not optimized for response time in very large repos, and that models weaker than those aider knows may have the map disabled by default (force it with `--map-tokens 1024`).

## Git integration is on by default

Aider is tightly integrated with git. In a git repo:

- `--auto-commits` (default True): aider commits its edits with a descriptive message.
- `--dirty-commits` (default True): before editing a dirty file, aider first commits the preexisting changes, keeping your work separate from aider’s edits.
- `/undo` reverts the last aider commit; `/diff` shows changes since the last message; `/commit` commits dirty changes; `/git` runs raw git commands (output excluded from the chat).
- Commit messages come from the weak model, given the diffs and chat history, following Conventional Commits by default; `--commit-prompt` customizes the prompt.

`--git-commit-verify` (default False) controls pre-commit hooks: by default aider commits with `--no-verify`, skipping them. `--no-git` stops aider from using git entirely; the docs recommend against it and note you should keep sensible backups. Without git there are no aider commits to undo.

## Commit attribution

Aider marks commits it authors or commits:

- If aider authored the changes, “(aider)” is appended to the git author and committer names; if it merely committed dirty changes, to the committer name. `--attribute-author` / `--attribute-committer` (both default True) control this.
- `--attribute-co-authored-by` (default True) instead appends a `Co-authored-by` trailer and disables the name suffixes unless those options are explicitly set to True — explicit True on the name attributes overrides the co-authored-by behavior.
- `--attribute-commit-message-author` (default False) prefixes “aider: ” when aider authored the changes; `--attribute-commit-message-committer` (default False) prefixes all commit messages.

## State files aider writes

| File | Purpose |
|---|---|
| `.aider.chat.history.md` | Chat history (default; `--chat-history-file`) |
| `.aider.input.history` | Input prompt history (default; `--input-history-file`) |
| `.aider.llm.history` | Full LLM conversation log, only when `--llm-history-file` is set |

By default aider offers to add `.aider*` entries to `.gitignore` (`--gitignore`, default True). `--restore-chat-history` (default False) restores the previous chat on launch.

## Official sources

- [Git integration](https://aider.chat/docs/git.html)
- [Specifying coding conventions](https://aider.chat/docs/usage/conventions.html)
- [Repository map](https://aider.chat/docs/repomap.html)
- [Supported languages](https://aider.chat/docs/languages.html)
- [Usage tips](https://aider.chat/docs/usage/tips.html)
- [FAQ](https://aider.chat/docs/faq.html)
- [Options reference](https://aider.chat/docs/config/options.html)
