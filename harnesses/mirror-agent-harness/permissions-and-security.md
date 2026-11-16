# Permissions and security

**Verified:** 2026-11-15

## Approval modes

“By default Mirror runs in **manual** mode: it pauses before shell commands (including ones that only read, such as `ls`), file writes and edits, connected-service tools, and most tools that extensions add. It shows the call, and you press `1` or `Enter` to approve, or `2` to reject. Reading and searching files with the agent's built-in tools never needs approval. Pressing `Escape` at an approval prompt stops the whole turn, not just that call.”

“Switch to **auto** mode to let the agent use tools without routine approval prompts. Tools that require explicit confirmation still ask, and external extensions need consent before loading.” Three ways to switch: “Press `F2` to toggle between manual and auto”, “Run `/approvals` to cycle between manual and auto”, or “Start Mirror with `mirror --approval-mode auto`.” In lists and prompts, “In a tool-approval prompt, `1` approves and `2` rejects, and `Enter` approves unless you select **Reject**.”

Scope and persistence: `--approval-mode auto|approve` — “Use automatic or manual tool approval, and remember the choice. Defaults to `approve`, which is manual.” “Each workspace remembers its own model, reasoning effort, and approval mode. … An explicit `--approval-mode` overrides the launch workspace's saved mode. Otherwise, that workspace keeps its own preference, falling back to the approval mode in `~/.mirror/config.json`.” Tool calls from connections “need your approval when the workspace uses manual approval.”

## Not a sandbox

The docs state the boundary twice: “Approval mode decides what you review. It isn't a sandbox. Approved commands, extensions, and local MCP servers run with your account's full permissions,” and the repo README repeats, “By default, tools run with your account's permissions. Manual approval is not a sandbox.” One mitigations the CLI does apply: “Mirror does keep its own provider API keys, such as `REFLECTION_API_KEY`, out of the environment of commands the agent runs” — the reference lists `REFLECTION_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`, `GEMINI_API_KEY`, `YOU_API_KEY`, `TAVILY_API_KEY`, and `FIRECRAWL_API_KEY` as variables agent-run commands don't receive. A local MCP server, by contrast, “runs with your account's permissions. Only configure servers you trust.”

## Remembered approvals and folder trust

`~/.mirror/tool_approvals.json` holds “Remembered tool approvals. Stop Mirror before editing it to revoke a rule.” First launch shows a “Using Mirror” notice, and “In each new directory, it then asks `Continue and trust this folder? [y/N]`”; for `mirror exec` and `mirror acp`, Mirror “requires the directory's notices to have been accepted interactively.”

## Extension consent

Consent is deliberately separate from tool approval: “In manual approval mode, each use of the tool needs your approval. External extensions also need consent before their code is imported, including in automatic approval mode. Read the code before enabling it.” The consent list offers “**Approve selected**, **Approve all**, or **Reject all**” (Approve selected “disables the other entries in that batch”), and “Press `Escape` to skip the batch for this launch. Mirror remembers approval and rejection choices.” The warning: “An extension is Python code that runs inside Mirror with your account's permissions. Approval mode doesn't restrict it. A repository you clone can include extensions in `.mirror/extensions/`, which is why Mirror asks first.” `--trust-extensions` enables “newly found extensions without being asked”; “Extensions you've already disabled stay disabled.” `MIRROR_EXTENSIONS_ENABLED=false` skips “all extensions outside Mirror's bundled ones for a session.”

## Sandbox workspaces

The one execution-isolation surface is `/sandbox`: “Its file and shell tools run in a microVM; extensions and local MCP servers still run on the host.” Sandboxes “require macOS on Apple Silicon, or Linux with access to `/dev/kvm`.”

## Usage analytics

“Mirror sends product usage events to Statsig so Reflection can see how its features are used. An event records that something happened, such as a submitted prompt, a slash command, a tool call, or a finished compaction, with counts and durations.”

| Sent | Never sent |
|---|---|
| “A random install ID created on first run and a session ID, on every event. The conversation ID, on conversation and turn events.” | “Prompts and model responses.” |
| “The Mirror version, OS, CPU architecture, locale, timezone, and terminal program.” | “Code and file contents.” |
| “Names such as the slash command or tool used, and a prompt's length.” | “Tool arguments and output.” |

“To turn analytics off, open `/settings` and switch **Usage analytics** off, or set `\"analytics\": false` in `~/.mirror/config.json`. Events that couldn't be sent are saved in `~/.mirror/analytics/` and retried the next time Mirror starts with analytics on, which also drops any older than five days.”

## Logs and feedback

“`/feedback`, to open a public GitHub issue form after confirmation. Nothing is attached or submitted automatically.” For bug reports: “If you attach logs from `~/.mirror/logs/`, review them first and remove anything private. Issues are public,” and “Don't share logs recorded with `--log-level debug`: they include request headers, which contain your API key.”

## Official sources

- [Mirror CLI overview](https://developers.reflection.ai/mirror-overview.md) · [CLI reference](https://developers.reflection.ai/mirror-cli-reference.md) · [Extensions](https://developers.reflection.ai/mirror-extensions.md) · [Connections and MCP](https://developers.reflection.ai/mirror-mcp.md) · [Workspaces](https://developers.reflection.ai/mirror-workspaces.md) · [Coding-agents quickstart](https://developers.reflection.ai/coding-agents-quickstart.md) · [GitHub: reflection-oss/mirror](https://github.com/reflection-oss/mirror)
