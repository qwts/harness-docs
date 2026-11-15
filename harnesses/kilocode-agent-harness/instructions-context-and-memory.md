# Instructions, context, and memory

**Verified:** 2026-11-15

## AGENTS.md

“AGENTS.md is an open standard for configuring AI agent behavior in software projects” — “a simple Markdown file placed at the root of your project.” Kilo loads it automatically and it “cannot be individually disabled — it is always loaded if present.”

- **Filenames:** `AGENTS.md` (“uppercase, plural - recommended”) with `AGENT.md` as “uppercase, singular - fallback.” The filename “must be uppercase (`AGENTS.md`), not lowercase (`agents.md`).”
- **Per-directory files:** subdirectory `AGENTS.md` files are “dynamically loaded when the agent reads files in that directory - they are not pre-loaded at session start”; contents are “injected into the conversation as `<system-reminder>` tags.”
- **Write protection:** “Both `AGENTS.md` and `AGENT.md` are write-protected files in Kilo Code” — the agent cannot modify them “without explicit user approval.”
- **Global fallbacks:** `~/.config/kilo/AGENTS.md` for Kilo, and “Claude-compatible: `~/.claude/CLAUDE.md`, until Claude Code Migration has been attempted. Migration ends this global fallback; project-level `CLAUDE.md` files keep working.”
- **Compatibility files:** project roots may carry `AGENTS.md` (primary), `CLAUDE.md` (“also supported for compatibility”), and `CONTEXT.md` (“additional project context”).
- **Legacy:** the old memory-bank feature is deprecated in favor of AGENTS.md; “Existing memory bank rules will continue to work.”

Instruction priority (new platform): agent prompt (`agent.<name>.prompt`) → project `instructions` key → project-root `AGENTS.md` → global `instructions` key → global fallbacks. Skills (`.kilo/skills/`, config `skills` key) are “loaded on demand.” The `instructions` config key takes file paths and glob patterns (e.g. `"[".kilo/rules/formatting.md", ".kilo/rules/*.md"]`); URL entries are fetched with a 5-second timeout and silently skipped on failure. `KILO_DISABLE_EXTERNAL_SKILLS=true` disables external skill directories (`.claude/skills/`, `.agents/skills/`).

## Custom rules

Rules are plain Markdown files — the migration guide stresses “Simple format: Plain Markdown files—no YAML frontmatter or GUI configuration required.” Convention: `.kilo/rules/*.md` in the project, `~/.kilo/rules/*.md` globally, with mode-specific behavior achieved through separate directories or custom modes rather than per-file metadata. Rules only take effect when referenced from the `instructions` array in `kilo.jsonc`.

## Memory

Memory is **opt-in** and **project-only**. Data lives under `~/.local/share/kilo/memory/<project-slug>-<sha1-12>/` with `project.md`, `environment.md`, `corrections.md`, plus a `sessions/` directory of digests. Subcommands: `/memory on|off|status|show|remember|correct|forget|auto|inspect|rebuild|purge` (destructive ones gated behind confirmation). Two agent tools back it — `kilo_memory_save` and `kilo_memory_recall` — which prompt for approval by default (recall can be allow-always). A `memory_model` slot (Kilo/auto/small tier options) processes saves and recalls; auto-save is best-effort with redaction. The older memory-bank feature is deprecated in favor of AGENTS.md.

## Context condensing

Compaction summarizes history when the window fills. Kilo “checks provider-reported usage after each response and estimates the outgoing text, system instructions, and tool definitions before contacting the provider. Compaction runs when either count reaches `compaction.threshold_percent`, or when the remaining window hits the reserved safety buffer, whichever happens first.”

| Setting | Behavior |
|---|---|
| `compaction.auto` | Auto-compaction on/off (default on) |
| `compaction.threshold_percent` | “Optional percentage from 1 to 100. Auto-compaction runs when token usage reaches this share of the model input or context window, unless the reserved safety buffer triggers first.” |
| `compaction.reserved` | “Token headroom reserved for the next turn. Applies only to models that advertise a separate input limit; models with a single context window use their full output cap as the reserve instead.” Default ~20,000 |
| `compaction.prune` | “Between turns, Kilo also runs a lighter prune pass. It walks completed tool outputs outside a 40,000-token recency window and replaces them with `"[Old tool result content cleared]"`.” |
| `compaction.tail_turns` | Recent turns kept verbatim (default 2) |

Slash command: “type `/compact` in chat (also findable by typing `smol` or `condense`).” Environment overrides: `KILO_DISABLE_AUTOCOMPACT=1` forces `compaction.auto = false`, `KILO_DISABLE_PRUNE=1` forces `compaction.prune = false`, and `KILO_EXPERIMENTAL_OUTPUT_TOKEN_MAX` overrides the 32,000-token default output ceiling. Models without a declared context window are not tracked for compaction.

## Codebase indexing

Semantic indexing is **opt-in and disabled by default**. Parsing uses Tree-sitter; the default vector store is LanceDB (embedded), with Qdrant as a remote alternative. Embedding providers include OpenAI, Ollama, OpenAI-compatible endpoints, Gemini, Mistral, Vercel, Bedrock, OpenRouter, and Voyage. Tunables: `searchMinScore` 0.4, `searchMaxResults` 50, `embeddingBatchSize` 60, 1MB max file size. Once indexed, the `semantic_search` tool (required `query`, optional `path`) “searches through your indexed codebase using semantic similarity rather than exact text matching,” returning “code snippets with file paths, line numbers, and similarity scores.”

## Ignoring files

The primary access-control mechanism is the permission system; `.kilocodeignore` remains supported via migration: “The IgnoreMigrator automatically converts `.kilocodeignore` patterns into permission deny rules on read and edit tools.” On top of that, two hardcoded layers always apply: “Hardcoded directory ignores — 27 directories are always skipped (e.g. `node_modules`, `.git`, `dist`, `build`, `.cache`, `__pycache__`, `vendor`, and others)” and “Hardcoded file pattern ignores — 11 file patterns are always skipped (e.g. lock files, binary artifacts).” File-watcher exclusions are a separate `watcher.ignore` setting.

## Official sources

- [AGENTS.md](https://kilo.ai/docs/customize/agents-md) · [Custom instructions](https://kilo.ai/docs/customize/custom-instructions) · [Custom rules](https://kilo.ai/docs/customize/custom-rules) · [Memory](https://kilo.ai/docs/customize/context/memory) · [Context condensing](https://kilo.ai/docs/customize/context/context-condensing) · [Codebase indexing](https://kilo.ai/docs/customize/context/codebase-indexing) · [Ignoring files](https://kilo.ai/docs/customize/context/kilocodeignore) · [Migrating from Cursor/Windsurf](https://kilo.ai/docs/getting-started/migrating)
