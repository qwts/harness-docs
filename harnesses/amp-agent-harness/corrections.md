# Amp corrections log

**Verified:** 2026-11-15

| Claim | Verdict | Correction |
|---|---|---|
| Amp asks for approval before running terminal commands. | ❌ FALSE | “By default, Amp does not ask for approval before running tools.” Control comes from plugins, MCP permission rules and per-tool policies, and enterprise enforcement — not an approval prompt. |
| Amp has allowlists/denylists for shell commands in its settings. | ❌ FALSE | No command allowlist setting is documented. `amp.tools.disable` disables whole tools by name; command-level policy is written as a plugin intercepting `tool.call` with allow/reject/modify/synthesize. |
| Amp is a VS Code extension. | ❌ FALSE | Amp is a web app, native apps (macOS, Windows, Linux, iOS, Android), and a CLI. IDEs attach to a CLI session via `ide connect`; no IDE extension is documented. |
| Threads are stored on your machine like local chat logs. | ❌ FALSE | Every thread has an `ampcode.com/threads/T-…` URL and opens across web, CLI, and apps; “a thread is not tied to the device you started it on,” and execution location is separate from viewing. |
| The Oracle is a built-in tool that edits files. | ❌ FALSE | The oracle is an optional plugin (`@amp/oracle`) that “reviews code and advises on hard decisions, and it never edits files”; it also expects the finder and librarian plugins. |
| Workspace settings are overridden by user settings. | ❌ FALSE | “Workspace settings override user settings”; only keymap entries in user settings override the workspace. |
| Settings keys use plain names like `model:` or `mcpServers:`. | ❌ FALSE | All settings keys use the `amp.` prefix (`amp.mcpServers`, `amp.showCosts`, …). |
| Amp commits are unattributed. | ❌ FALSE | By default Amp adds an `Amp-Thread-ID: <thread-url>` trailer and a `Co-authored-by: Amp <amp@ampcode.com>` co-author; both are settings-controlled. |
| `AMP_API_KEY` is just another saved login. | ❌ FALSE | It “takes precedence over saved accounts until you unset it,” and execute-mode tokens must start with `sgamp_`; `amp login` session tokens are rejected because they expire within an hour. |
| Amp runs entirely on your machine. | ❌ FALSE | Amp sends model requests from its own servers (providers must allow its egress IPs), remote MCP definitions are stored by ampcode.com, and threads can execute in orbs or on runners. |
| MCP servers configured in `.amp/settings.json` start automatically when you open a project. | ❌ FALSE | Workspace-configured servers “require explicit approval before running”; pending servers surface in `amp mcp doctor` and are approved with `amp mcp approve`. |
| `amp.mcpPermissions` blocks any server it does not list. | ❌ FALSE | “Amp applies the first matching rule” and unmatched servers are allowed; blocking everything requires explicit reject rules (e.g. `{"command": "*"}` and `{"url": "*"}`). |
| Skills live in `.amp/skills/`. | ❌ FALSE | The documented locations are `.agents/skills/` in projects, `~/.config/agents/skills/`, `~/.agents/skills/`, `~/.config/amp/skills/`, `.claude/skills/`, `~/.claude/skills/`, `~/.claude/plugins/cache/`, `amp.skills.path`, and the personal/workspace/official repositories; no `.amp/skills/` path is documented. |
| Amp cannot use Claude Code skills. | ❌ FALSE | Claude-compatible locations (`.claude/skills/`, `~/.claude/skills/`, `~/.claude/plugins/cache/`) are searched by default, with `amp.skills.disableClaudeCodeSkills` to opt out. |
| Plugins sandbox the agent from dangerous actions. | ⚠️ PARTIAL | Plugins can intercept and reject tool calls before they run, but they “run code in your environment”; for untrusted inputs the docs recommend a policy plugin or an isolated development environment, not plugins as a sandbox. |
| Amp works with local models over localhost (e.g. a local Ollama). | ❌ FALSE | “Amp sends model requests from its own servers,” so callers must allow Amp’s egress IPs; documented connection types are cloud API keys, subscriptions, cloud platforms, and gateways — no localhost endpoint. |
| The model selector can change a plugin agent’s model. | ❌ FALSE | A custom agent’s model comes from `model` in `amp.createAgent(...)`; “Model Selector settings cannot change a plugin agent’s model.” |
| Subagents can talk to each other and share their work live. | ❌ FALSE | Subagents “work in isolation”: no inter-subagent communication, no direct guidance, and the main agent sees only the final summary. |
| Subtree AGENTS.md files load at session start. | ❌ FALSE | Cwd and parent files (to `$HOME`) always load; “subtree AGENTS.md files are included when the agent reads a file in the subtree.” |
| Amp reads CLAUDE.md before AGENTS.md. | ❌ FALSE | AGENTS.md is the primary file; `CLAUDE.md` (or `AGENT.md`) is a fallback included only where no `AGENTS.md` exists. |
| Workspace AGENTS.md guidance reaches subagents too. | ❌ FALSE | Workspace guidance “does not apply to subagents, Puck, or raw custom agents.” |
| `amp -x` always fires the plugin lifecycle events. | ⚠️ PARTIAL | Execute mode starts turns immediately unless `--plugin-ready-timeout` waits for plugins; without it, `agent.start` and `agent.end` events can be skipped. |
| Orbs and runners are the same thing. | ❌ FALSE | An orb is “a machine Amp creates” (a fresh cloud sandbox per thread); a runner “runs on a machine you already have,” and every thread on it shares that one machine. |
| Pausing or snoozing a thread stops the agent and its automations. | ❌ FALSE | Snoozing “does not stop the agent, and it does not pause the thread’s automation; scheduled runs keep firing and using credits.” |
| Archiving a thread deletes it. | ❌ FALSE | Archiving removes the thread from active lists; “the URL still works and the agent can still reference it.” Delete is a separate action that removes it permanently. |
| Multiplayer works on any shared thread. | ❌ FALSE | “Multiplayer is not available while anyone with the link can view the thread.” |
| Minimal data retention means Amp never retains data. | ❌ FALSE | The docs are explicit: “It does not mean that no data is ever retained” — safety-monitoring data and some model flows (up to 30 days where documented) are exceptions, and MDR “does not control how Amp stores thread data.” |
| Workspace admins can disable every model, including the default. | ❌ FALSE | The new-thread default model and the models Puck and the Spaces agent run on are locked and cannot be disabled. |
| The enterprise MCP registry allowlist is advisory. | ❌ FALSE | The machine that starts a server checks it against the registry, and “if a machine cannot reach the registry, it blocks every MCP server it would start”; package workarounds (`--from`, `--with`, alternate indexes, `NODE_OPTIONS`) are documented as blocked. |
| Amp has no Windows support. | ❌ FALSE | Native Windows (and iOS/Android) apps are documented; the CLI specifically supports Windows “through WSL.” |
| The Amp SDK is TypeScript-only. | ❌ FALSE | The SDK “allows you to programmatically use the Amp agent in your TypeScript or Python programs.” |
| Staff Pick is always the optimal model choice. | 💬 OPINION | The docs describe it as “the model the Amp team would choose for most work”; model choice is presented as workload-dependent, not a ranking. |
| Amp requires manual reinstalls to update. | ❌ FALSE | The CLI updates in the background — “You don’t have to do anything except regularly restart Amp” — with `amp update` as the manual path and `amp.updates.mode` to change behavior. |
