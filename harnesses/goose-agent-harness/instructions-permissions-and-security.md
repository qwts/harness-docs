# Instructions, permissions, and security

**Verified:** 2026-10-29

## Instruction discovery

Goose’s default context filenames are `AGENTS.md` and `.goosehints`. `CONTEXT_FILE_NAMES` accepts a JSON array to replace that default with one or more filenames, including compatibility names such as `CLAUDE.md`.

The documented behavior requires the Developer extension:

- Global hints default to `~/.config/goose/.goosehints`.
- Project hints apply to the current directory hierarchy.
- At startup, goose loads applicable files from the working directory up through the repository root.
- As goose accesses nested directories, it can discover additional context files there.
- Newly discovered nested instructions remain active for the rest of the session; restart after edits for reliable reload.
- Local instructions take priority over conflicting global preferences.
- `@path` includes referenced file content in immediate context; a plain path tells goose to consult it when needed.

Hints enter the model’s system context. They can direct workflow, but they are not permission enforcement or a filesystem boundary.

## Persistent instructions are model-observed context

Persistent instructions use the Model-Observed Internal Memory (MOIM) path:

- `GOOSE_MOIM_MESSAGE_TEXT` injects literal text every turn.
- `GOOSE_MOIM_MESSAGE_FILE` injects a file’s contents every turn.
- If both are set, their contents are concatenated.
- Content is capped at 64 KB with UTF-8-safe truncation.

Unlike startup hints, these values are reread and injected each turn, so changes can take effect without restarting. They are useful for prominent reminders, but “persistent” describes context delivery, not a deterministic control that hostile model output cannot bypass. Use actual permission, hook, process, container, or operating-system controls for enforcement.

## Permission modes

| Mode | CLI value | Verified behavior |
|---|---|---|
| Completely Autonomous | `auto` | File changes, extension use, and deletion without approval; **default** |
| Manual Approval | `approve` | Confirmation before applicable tools/extensions; supports per-tool overrides |
| Smart Approval | `smart_approve` | Automatically approves actions classified as low risk and requests approval for others |
| Chat Only | `chat` | No extension use or file modification |

Desktop sessions each have a mode; changing the default affects new sessions, not existing ones. The CLI can change mode with `/mode` or save a default through `goose configure`.

In Manual and Smart modes, the documented approval flow focuses on tools classified as writes, such as edits and state-changing shell commands. Read/write classification is a **best-effort interpretation by the LLM provider**, not a deterministic parser or proof that every side effect will be caught.

### Per-tool permissions

Tool permissions supplement Manual and Smart modes:

- **Always Allow** — run without approval.
- **Ask Before** — require confirmation.
- **Never Allow** — deny use of that tool.

These rules control tool invocation, not what an allowed program does after launch. Tool availability is separately determined by enabled extensions, `available_tools`, provider type, and subagent/recipe configuration.

## Default authority and sandbox boundary

The Developer extension is enabled by default and supplies shell and file tools. With the default autonomous mode, goose can run commands with the launching user’s privileges and modify any file that user can access without approval. Shell commands inherit the goose process environment, which can include credentials; Desktop launches may have a different environment from an interactive shell.

The current core documentation does **not** establish a general OS-level filesystem/network sandbox around these default Developer tools. Chat Only removes tool use, and approval/hook controls can deny calls, but neither is equivalent to OS confinement. Sandboxes belonging to an external CLI provider or a container are properties of that external execution environment, not proof that ordinary goose API-provider sessions have the same boundary. MCP Apps’ sandboxed display windows are also a separate UI mechanism.

## Prompt-injection detection

Detection is disabled by default and enabled with `SECURITY_PROMPT_ENABLED` or Desktop settings.

- Pattern matching inspects tool-parameter text before execution.
- Optional ML classification also analyzes semantic tool-call content and recent messages.
- Findings above the configured threshold pause for **Allow Once** or **Deny**.
- An allowed command still runs with the user’s full permissions.
- The default threshold is `0.8`; the docs describe higher thresholds as stricter/more alerts.
- ML classification sends tool-call content and recent messages to the configured endpoint, creating a separate privacy consideration.

This is a safeguard for recognized or inferred threats, not a guarantee against novel attacks.

## Adversary mode

Creating `~/.config/goose/adversary.md` enables an independent model reviewer; deleting the file disables it. An empty file uses default rules. The reviewer sees the original task, recent messages, and selected tool-call details, then returns ALLOW or BLOCK.

- It reviews `shell` by default.
- A `tools:` line can expand the selected tool set.
- It uses the same provider and model as goose.
- **Reviewer failure allows the tool call: fail-open.**

A second model can add contextual review, but it is not a deterministic policy engine and does not cover every tool unless configured.

## Hooks and allowlists are separate controls

`PreToolUse` plugin hooks can deterministically block selected calls. Their default failure mode is allow; use `on_failure: block` for a required runtime policy, while recognizing that configuration-loading and matcher errors are not thereby made fail-closed. Other hook events are mostly observational. See [MCP, hooks, plugins, and integrations](mcp-hooks-plugins-and-integrations.md).

`GOOSE_ALLOWLIST` points to a remotely fetched YAML list of exact extension installation commands. It restricts installation, not the behavior of an allowed extension, arbitrary tool execution, or manual changes outside the covered installation path. Use HTTPS and protect both the list and local configuration from unauthorized modification.

## Practical control composition

For sensitive work, combine least-privilege process credentials and an actual container/VM with:

1. Chat Only or Manual Approval where appropriate.
2. Narrow extension and tool availability.
3. Explicit per-tool denials.
4. Reviewed `PreToolUse` hooks with deliberate failure behavior.
5. Prompt-injection and adversary checks as additional, fallible layers.
6. Review of recipes, plugins, hooks, MCP commands, install scripts, and remote endpoints before trust.

## Official sources

- [Hints and custom context files](https://goose-docs.ai/docs/guides/context-engineering/using-goosehints)
- [Persistent instructions](https://goose-docs.ai/docs/guides/context-engineering/using-persistent-instructions)
- [Permission modes](https://goose-docs.ai/docs/guides/managing-tools/goose-permissions)
- [Tool permissions](https://goose-docs.ai/docs/guides/managing-tools/tool-permissions)
- [Developer extension and access controls](https://goose-docs.ai/docs/mcp/developer-mcp)
- [Prompt-injection detection](https://goose-docs.ai/docs/guides/security/prompt-injection-detection)
- [Adversary mode](https://goose-docs.ai/docs/guides/security/adversary-mode)
- [Extension allowlist](https://goose-docs.ai/docs/guides/allowlist)
- [Hooks](https://goose-docs.ai/docs/guides/context-engineering/hooks)
