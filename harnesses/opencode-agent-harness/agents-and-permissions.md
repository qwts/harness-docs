# Agents and permissions

## Agent types

- **✅** Two kinds: **primary** (you talk to it; Tab cycles them) and **subagent** (invoked by primary agents by description match, or manually via `@name` mention).

## Built-in agents

| Agent | Mode | Notes |
|---|---|---|
| `build` | primary | Default; all tools enabled |
| `plan` | primary | Read-only planning; `edit` + `bash` default `ask` |
| `general` | subagent | Full tool access (except todo); for parallel work |
| `explore` | subagent | Fast read-only codebase exploration |
| `scout` | subagent | Read-only external docs/dependency research (clones deps into OpenCode cache) |
| `compaction` | primary (hidden) | Auto context compaction; not selectable |
| `title` | primary (hidden) | Auto session titles |
| `summary` | primary (hidden) | Auto session summaries |

## Defining agents

Two ways, merge into one namespace:

- **JSON**: `agent.<name>` in opencode.json — `{ description (required), mode: "primary"|"subagent", model, temperature, prompt: "{file:...}", steps, disable, permission, tools (deprecated) }`
- **Markdown**: frontmatter + body = prompt. `.opencode/agents/review.md` → `@review`; global `~/.config/opencode/agents/`.
- **✅** Subagents inherit the invoking agent's model unless `model` set; primary agents use global `model`.
- **✅** `steps` caps agentic iterations (legacy `maxSteps` deprecated).
- **✅** Default temperature is model-specific (~0 for most, 0.55 for Qwen) if unset.

## Permission model

- **✅** Replaces the deprecated `tools` boolean map since v1.1.1 (old `tools` still parses: `true` ≈ `{"*":"allow"}`, `false` ≈ `{"*":"deny"}`).
- Actions: `"allow"`, `"ask"`, `"deny"`. `permission` can be a bare string (`"allow"`), a map keyed by tool, or granular objects per tool.
- **✅** Granular rules use `*`/`?` wildcards; **last matching rule wins** — put `"*"` first.
- `~`/`$HOME` expansion works at pattern start.

### Permission keys

`read`, `edit` (covers edit/write/patch), `glob`, `grep`, `bash` (matches parsed command, so `git *` covers `git status`), `task` (subagent type), `skill`, `lsp`, `question`, `webfetch`, `websearch`, `external_directory`, `doom_loop`.

### Defaults

- **✅** Most default `allow`; `external_directory` and `doom_loop` default `ask`.
- **✅** `read` defaults allow **except** `*.env`/`*.env.*` denied, `*.env.example` allowed.
- `external_directory` gates any tool touching paths outside the working dir; allowed dirs inherit workspace defaults.
- `doom_loop` fires when the same tool call repeats 3x with identical input.

### "Ask" UX

Prompt offers `once` / `always` (session-scoped, pattern suggested by the tool e.g. `git status*`) / `reject`.

### `--auto` mode

- **✅** `opencode --auto` / `opencode run --auto` auto-approves anything not explicitly `deny`-ed; TUI command palette can toggle ("auto" indicator shows in prompt). Explicit denies still enforced.

### Per-agent permissions

- **✅** `agent.<name>.permission` merges over global; agent rules take precedence. Same object syntax in markdown frontmatter.
