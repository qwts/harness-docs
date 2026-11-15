# Memory, taste, and sessions

**Verified:** 2026-11-15

## Memory is AGENTS.md

Memory “lives in plain markdown files named AGENTS.md.” Three tiers, and “every one that exists is loaded; they add up, they don’t replace each other”:

| Tier | Path |
|---|---|
| User | `~/.commandcode/AGENTS.md` |
| Project | `<project>/AGENTS.md`, else `<project>/.commandcode/AGENTS.md` (first that exists, not both) |
| Subdirectory | `<subdir>/AGENTS.md`, else `<subdir>/.commandcode/AGENTS.md` — loaded on demand when a file there is read, walking up to the project root outermost-first |

“Command Code reads AGENTS.md, not CLAUDE.md”; `/init` writes a starter project AGENTS.md and `/import` migrates from other agents, rewriting references. `@path` references are replaced with file contents — relative to the importing file, recursive “up to 5 levels deep,” ignored inside code fences, and left as plain text if unresolvable. Memory is part of the system prompt: “It’s re-read every request,” “It survives compaction,” and “It costs tokens on every turn” (`/context` shows the cost).

## Taste: learned preference rules

Taste is Command Code’s learned-preference layer, powered by what the docs call “meta neuro-symbolic AI model taste-1 with continuous reinforcement learning” — and they are explicit that “taste-1 is not an LLM. It does not generate text and it does not answer prompts”; it classifies signals (from “every accept, reject, and edit” plus “correction diffs from your commits”) into categories with a 0–1 confidence. An LLM writes the resulting learnings as markdown, stored by the CLI at `.commandcode/taste/` (project), `~/.commandcode/taste/` (global), or `commandcode.ai/username/taste` (remote, shared via `npx taste push`/`pull` — “npx taste and cmd taste are the same command”). “Taste learning runs locally and stores preferences as structured rules — not code snippets.”

Controls: `tasteLearning` in settings (project `settings.local.json` takes precedence), `cmd taste disable [--user]`, and `taste lint` validates markdown before push. taste-1 is “Included in every plan at no extra charge”; the two LLM steps (input processing, writing learnings) bill normally and are model-selectable via `featureModels.tasteLearning`/`tasteOnboarding`.

## Sessions: append-only JSONL trees

“Every conversation … is a session: a durable transcript on disk that you can resume, rename, fork, clone, rewind, share, and export.” Sessions are per-project at `~/.commandcode/projects/<project-slug>/<session-id>.jsonl` — “a single append-only JSONL file” whose entries form a tree (each entry points at a parent; “the file is never rewritten in place”), committed per turn. Sidecars carry meta, share, checkpoints, and prompts. `/resume` and `--continue` “only ever see sessions from the directory you launched Command Code in”; `--no-session` keeps a run in-memory and hidden; `--fork-session` forks on resume; `/fork` copies the whole tree while `/clone` copies only the active branch (no checkpoints); `/rewind` and double-Esc (within 500 ms) restore checkpoints.

Checkpoints: “Every prompt you send opens a checkpoint: a snapshot of the conversation plus a backup of every file Command Code goes on to edit.” Restore modes are Both, Conversation only, and Code only; restores are all-or-nothing with pre-verified backups. “Checkpoints are kept for 30 days, capped at 200 backups per session,” file backups “capped at 10MB per file,” stored under `~/.commandcode/file-history/<session-id>/`.

Compaction (automatic or `/compact`) appends an entry to the tree — “The on-disk log is never rewritten” — and full history stays reachable in `/tree`. A per-session scratchpad at `/tmp/commandcode-<uid>/…/scratchpad` is permission-prompt-free, shared with sub-agents, and relocatable via `COMMANDCODE_SCRATCHPAD_BASE` (`"disableScratchpad": true` turns it off). Export paths: `/share gist [html|jsonl|md]` (secret GitHub gist via `gh`), `/export [md|html|jsonl]`, `/copy`; the link-sharing `/share` “is being rebuilt.”

## Official sources

- [Memory](https://commandcode.ai/docs/memory)
- [Taste](https://commandcode.ai/docs/taste)
- [Sessions & Checkpoints](https://commandcode.ai/docs/sessions)
- [Context & Compaction](https://commandcode.ai/docs/context)
- [Settings & Config](https://commandcode.ai/docs/settings)
