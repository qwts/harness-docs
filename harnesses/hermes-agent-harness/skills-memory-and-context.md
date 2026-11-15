# Skills, memory, and context files

**Verified:** 2026-11-15

## The file model

| File | What it holds | Who writes it | When the agent sees it |
|---|---|---|---|
| `SOUL.md` | Personality, tone, identity | You (seeded if absent) | Slot #1 of the system prompt, session start — from `HERMES_HOME` only |
| `USER.md` | User profile | The agent (`memory` tool) | Frozen snapshot at session start |
| `MEMORY.md` | Environment facts, lessons | The agent (`memory` tool) | Frozen snapshot at session start |
| `.hermes.md` / `HERMES.md` | Project instructions, highest priority | You | Session start (first match wins) |
| `AGENTS.override.md` | Personal per-directory AGENTS override (gitignored) | You | Instead of the sibling `AGENTS.md` |
| `AGENTS.md` | Project instructions | You | Session start + progressive discovery |
| `CLAUDE.md`, `.cursorrules`, `.cursor/rules/*.mdc` | Claude/Cursor compat context | You | Lower-priority fallbacks |

“Only **one** project context type is loaded per session, first match wins”: `.hermes.md` → `AGENTS.override.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules`. Inside a git repo, a merged chain of `AGENTS.md` files loads from the git root down to the working directory — “Deeper files appear later in the prompt, so more specific guidance takes precedence”; outside a repo, “only the working directory itself is checked — parents are never consulted.”

## Context loading

Startup pipeline: scan for the first-match file → read (symlinks resolved; targets must stay inside the project, or inside the file’s own directory when the root is `$HOME` or above) → prompt-injection security scan → head/tail truncation (70%/20% with a 10% marker) → assembly under `# Project Context`. Per-file cap: `context_file_max_chars` when set, otherwise dynamic with the model window — “floor 20,000 chars, ceiling 500,000”; read timeout 5 s.

During the session, `SubdirectoryHintTracker` watches tool-call paths, checks each directory and up to 5 ancestors once per session, and appends discovered files to the tool result — capped at a fixed 32,000 characters per hint regardless of the startup cap.

## Memory

Two bounded stores in `~/.hermes/memories/`: `MEMORY.md` at 2,200 chars (~800 tokens) and `USER.md` at 1,375 chars (~500 tokens). The `memory` tool takes `add`/`patch`/`remove` operations against a top-level `target` (`"memory"` or `"user"`) — there is **no** `read` action (content is injected) and **no** `replace` action (removed; models used it as a destructive find-and-swap). `patch`/`remove` match short unique substrings. At the limit, the tool errors with the exact overage and the agent reissues one batch that frees space and adds the entry — “the limit is checked only on the batch’s final result.” Duplicates are rejected; entries are injection-scanned.

The system-prompt block is a frozen snapshot showing usage percentage and `§`-separated entries. The learning loop runs around session boundaries: “run `/new` at natural boundaries — a finished task, a change of topic, the start of a day.”

`memory.write_approval: true` gates every save: inline prompts in the interactive CLI, staged review (`/memory pending`, `/memory approve|reject <id>`) everywhere else, including the background self-improvement review. External providers (RetainDB and ByteRover bundled; Honcho, Hindsight, Supermemory, Mem0, OpenViking, Holographic as plugins) run **alongside** built-in memory; `memory.prefetch_spill_enabled` caps oversized recall at a head/tail preview.

`/journey` (aliases `/learning`, `/memory-graph`) renders the timeline of learned skills and memories; `hermes journey list|delete|edit` prunes and corrects it.

## Skills

Skills live in `~/.hermes/skills/` and follow “the agentskills.io open standard”: a `SKILL.md` with frontmatter (`name`, `description` ≤60 chars, `version`, `platforms: [macos|linux|windows]`, `metadata.hermes.tags/config`, conditional `requires_toolsets`/`fallback_for_toolsets`) plus `references/`, `templates/`, `scripts/`, `assets/`. Loading is progressive disclosure:

```
Level 0: skills_list()           → index (~3k tokens)
Level 1: skill_view(name)        → full SKILL.md
Level 2: skill_view(name, path)  → one reference file
```

Every installed skill becomes a `/<name>` slash command (up to 5 stackable per message); **skill bundles** (`~/.hermes/skill-bundles/<slug>.yaml`) group several skills under one command and win slug collisions. `/learn` authors skills from anything — directories, URLs, pasted notes, whole books (large sources become knowledge-base skills with per-chapter `references/` loaded on demand). The agent saves its own procedural memory through `skill_manage` (`create`/`patch`/`delete`/`write_file`/`remove_file`); an advisory linter warns on `incident-log-shape`, `references-sprawl` (>60 files), and `oversized-body` (~24k chars) but never blocks.

**Trust and scanning:** Hub sources are `official` (built-in trust), `skills-sh`, `well-known` (`/.well-known/skills/index.json`), `github` (default taps include openai/skills, anthropics/skills, huggingface/skills, NVIDIA/skills), `url`, plus ClawHub/LobeHub/browse.sh community registries. Every install passes the security scanner — “data exfiltration, prompt injection, destructive commands, supply-chain signals” — and `--force` “does **not** override a `dangerous` scan verdict.” An optional advisory NVIDIA SkillEvaluator Tier 1 scan (PII, unicode smuggling, script lint) prints findings but never blocks. Project-local skills (`<root>/.hermes/skills/` and `.agents/skills/`) require `hermes skills trust` and are content-hash re-scanned on change; a dangerous verdict quarantines the skill. Precedence: `project → local → skills.create_dir → external_dirs`; inside one directory, same-name different-skill collisions are ambiguous and refused, never guessed.

**Write gating:** `skills.write_approval: true` stages every write under `~/.hermes/pending/skills/` for `/skills diff|approve|reject`; `hermes curator lock <skill>` makes one skill read-only to the agent in every kind of session.

## Personality

`SOUL.md` is injected verbatim as slot #1; `/personality <name>` overlays a session-level preset (`helpful`, `concise`, `technical`, `creative`, `teacher`, `kawaii`, `pirate`, `noir`, … — custom personas in `personalities:` config). `/personality none` clears it.

## Official sources

- [Skills System](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) · [Persistent Memory](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory) · [Context Files](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files) · [Which File Does What?](https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what) · [Personality](https://hermes-agent.nousresearch.com/docs/user-guide/features/personality) · [Curator](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator)
