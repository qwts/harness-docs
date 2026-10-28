# Configuration and context

**Verified:** 2026-10-29 · **Surface:** Muse Code CLI/runtime

## Settings

Muse Code user settings live at:

```text
~/.config/muse/settings.json
```

This is the vendor-documented settings path. A separate organization's worktree or identity configuration is not evidence of a Muse Code settings location.

## Initialization and trust

`muse init` writes only `AGENTS.md`. It does not scaffold a collection of configuration files and does not modify `settings.json`.

Project instructions load only after the workspace is trusted. Trust therefore changes context ingestion as well as execution behavior. Running in an untrusted workspace is not equivalent to running with the repository's instruction files active.

## Instruction-file selection

For each directory, Muse Code selects instruction files in this order:

```text
1. AGENTS.md
2. CLAUDE.md
3. .agents/AGENTS.md
4. .claude/CLAUDE.md
```

This is a selection order per directory, not a promise to concatenate every same-scope alternative. Project rules override user rules, and deeper project rules override shallower project rules.

Practical consequences:

- Use the first supported file in the selection order when multiple exist and only one should be selected.
- Put directory-specific guidance deeper in the tree when it should refine project-wide guidance.
- Do not assume compatibility-named files have identical semantics in Muse Code and their originating harness.

## Shared project memory

Shared project memory uses:

```text
.agents/memory/MEMORY.md
.agents/memory/<topic-files>
```

The startup memory index covers up to 48 files. This is an indexed memory system, not an instruction to place arbitrary runtime state or secrets in the repository. Keep memory content appropriate for sharing with everyone who can access the project.

## Configuration review checklist

When context appears missing or wrong:

1. Confirm the workspace is trusted.
2. Check which instruction file wins selection in each relevant directory.
3. Check for deeper project rules that override shallower ones.
4. Distinguish project instructions from `.agents/memory/` content.
5. Confirm whether settings were expected to change; `muse init` does not edit them.

## Sources

- [Muse Code overview](https://dev.meta.ai/docs/muse-code)
- [Configuration](https://dev.meta.ai/docs/muse-code/configuration)
