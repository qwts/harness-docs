# Subagents and custom agents

**Verified:** 2026-11-15

## Task and Read Thread subagents

“Amp can hand focused work to a subagent with the Task tool.” A Task subagent “runs the same model and reasoning effort as your thread, in its own context window,” carries out delegated work end to end — it has file-editing and terminal tools — and reports a summary back. Task also carries “skills for read-only roles, which are searching code in your workspace, researching another repository or dependency, reviewing a change, and planning work.” A second subagent, Read Thread, “reads and summarizes other Amp threads.”

Amp decides on its own when to spawn a subagent, and users can request one explicitly (“ask the oracle”, “use the librarian” trigger the matching subagents when their plugins are installed). Good fits, per the docs: multi-step decomposable work, output-heavy operations, parallel tasks, and keeping the main thread’s context clean.

Subagent semantics worth knowing:

- Subagents “work in isolation”: they cannot talk to each other, receive no direct guidance, and start with only the context the main agent provides; the main agent sees final summaries, not step-by-step work.
- The main agent can hand a Task subagent a follow-up, which it processes “with its full earlier context” — or, if still working, as extra instructions for the current task.
- When a subagent’s context fills, Amp compacts it “the same way it compacts a thread.”
- Changing the thread’s model also changes later tasks for existing subagents, since subagents inherit the thread’s model and reasoning effort.

## Oracle, librarian, and finder are plugins

The reviewing trio ships as installable plugins, not built-ins:

| Plugin | Behavior |
|---|---|
| `@amp/oracle` | Adds the oracle tool: “reviews code and advises on hard decisions, and it never edits files” |
| `@amp/librarian` | “Clones repositories outside your workspace into /tmp and explains their code and history” |
| `@amp/finder` | “Searches your workspace and returns the relevant files and line ranges” |

Install with `amp plugins add`, then reload via `plugins: reload` or restart. If you install the Oracle, install the other two as well — “the Oracle hands searches to finder and librarian.” Each plugin pins its own model through `MODEL` and `REASONING_EFFORT` at the top of the plugin file (`amp plugins show-agent-options` lists choices), and edited plugin files are overwritten by updates unless `--auto-update` is avoided. Plugins install only into `~/.config/amp/plugins/` locally; to have them everywhere, including orbs, add them as personal plugins via URLs.

## Custom agents through plugins

Plugins can define custom agents — shown in the model selector under “Your Agents” — and expose custom subagents as tools (connected to the invoking thread via `parentThreadID`). A custom agent’s model is set by `model` in `amp.createAgent(...)`, its Task subagents use the same model, and “Model Selector settings cannot change a plugin agent’s model.” Built-in agent modes are `low`, `medium`, `high`, and `ultra`; the older `rush`, `smart`, and `deep` modes are deprecated aliases mapping to `low` and `medium`. Workspace guidance does not apply to subagents, Puck, or raw custom agents.

## Model and effort context

The model selector picks the main agent’s model and reasoning effort; Amp “builds the agent around the model you pick, with a system prompt and tools made for that model,” and Staff Pick gives the Amp team’s default choice. Supporting system models handle image/PDF/audio/video analysis, image generation, thread naming, and compaction — “a frontier model where deep reasoning matters and a faster specialist where speed matters more.” Note for BYOK users: “Anthropic requires data retention for Claude Fable models,” so Claude Fable connections require enabling data retention on the workspace owning the key.

## Official sources

- [Models and subagents](https://ampcode.com/docs/models-and-subagents)
- [Tools](https://ampcode.com/docs/tools)
- [Plugins](https://ampcode.com/docs/customize/plugins)
- [AGENTS.md](https://ampcode.com/docs/customize/agents-md)
