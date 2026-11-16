# Tools, permissions, and security

**Verified:** 2026-11-16 (Pi v1.5.2)

## Built-in tools

| Built-in | Purpose |
|---|---|
| `read` | “Read text files and supported images” |
| `bash` | “Run shell commands” |
| `powershell` | “Run PowerShell commands on Windows” |
| `edit` | “Apply exact text replacements to an existing file” |
| `write` | “Create or overwrite a file” |
| `grep` | “Search file contents” |
| `find` | “Find paths using glob patterns” |
| `ls` | “List directory contents” |

Two more come from built-in extensions and are “off by default; the MCP extension turns them on when an MCP server needs them”: `codemode` (“Run JavaScript that calls the other tools … only the script's output reaches the model”) and `tool_search` (“Search tools that are not declared to the model … and declare the matches for the next call”).

## Tool selection (the permission model)

There is no approval-mode system. “Pi shows each file read, search, command, and edit it performs. It does not ask before every tool call.” Runtime control is by allowlist:

- `-t, --tools <list>` — “Replaces the default selection with a comma-separated allowlist of built-in, extension, or custom tools. Entries are tool names or patterns where `*` matches any characters.” A list of only `+name`/`-name` entries “is not an allowlist; it changes the default selection instead” (e.g. `pi --tools +codemode,-write`). “Plain names and `+name`/`-name` entries cannot be mixed.”
- `-xt, --exclude-tools <list>` — “Disables comma-separated tool names or patterns after all other selection options, MCP tools included.”
- `-nbt, --no-builtin-tools` — “Disables default built-in tools while retaining extension and custom tools.”
- `-nt, --no-tools` — “Starts with all built-in, extension, custom, and MCP tools disabled.”

The `defaultTools` setting (default `read`, `bash`, `edit`, `write`) composes with project settings — “a project list with only `+name` and `-name` entries changes the user's selection, and a project list with a plain name replaces it.” `/reload` “enables tools newly added to `defaultTools` … a tool removed with `-name` stays removed.”

**MCP interaction:** “`--tools` selects the tools declared to the model. It does not remove MCP tools, whose reach is set by their exposure … An MCP tool that no entry names or matches is never declared directly, whatever its exposure; only `tool_search`, if listed, can load it. Once an entry starts with `mcp__`, `--tools` filters MCP tools too, so this keeps only the tools of the `radius` server” (`pi --tools read,bash,codemode,'mcp__radius__*'`). “To remove MCP tools, use `--exclude-tools 'mcp__*'` or `--no-mcp`.”

## Tool pipeline and permission gates

“Every MCP call passes through Pi's tool pipeline. Extension `tool_call` and `tool_result` handlers, including permission gates, therefore apply to MCP tools.” A gate blocks by returning `{ block: true, reason }` from a `tool_call` handler — the canonical example confirms destructive commands interactively:

```typescript
pi.on("tool_call", async (event, ctx) => {
  if (isToolCallEventType("bash", event) && event.input.command.includes("rm -rf")) {
    const ok = await ctx.ui.confirm("Dangerous!", "Allow rm -rf?");
    if (!ok) return { block: true, reason: "Blocked by user" };
  }
});
```

Gates can be annotation-driven: `pi.getAllTools()` reports MCP/extension tool annotations (“`readOnlyHint`, `destructiveHint`, `idempotentHint`, and `openWorldHint` … The hints are not verified, but a permission extension can use them to decide which calls to confirm”). Nested calls made through `ctx.executeTool()` go “through argument validation and the `tool_call` and `tool_result` handlers like model-issued calls.” Tool calls from one assistant message “run in parallel by default” unless a tool sets `executionMode: "sequential"`; file-mutating custom tools should use `withFileMutationQueue()` so it “participates in the same per-file queue as built-in `edit` and `write`.”

## Project trust

“Project trust controls whether Pi loads most settings and resources supplied by a working folder. It prevents a folder from silently loading executable extensions before you approve it.” Pi requires a decision when it finds any of: `.pi/settings.json`, `.pi/mcp.json`, `.pi/extensions`, `.pi/skills`, `.pi/prompts`, or `.pi/themes`, `.pi/SYSTEM.md` or `.pi/APPEND_SYSTEM.md`, and “project `.agents/skills` in the current directory or an ancestor directory” — “A bare `.pi` directory does not require project trust.”

Decision order: “A command-line `--approve` or `--no-approve` override applies first”; otherwise “The first extension that returns yes or no owns the decision” (`project_trust` event); then “a saved decision for the current directory or one of its parents” from `~/.pi/agent/trust.json` (saved via `/trust`); then “the global `defaultProjectTrust` setting, whose default is `"ask"`.” Headless behavior: “Print, JSON, and RPC modes cannot show the built-in trust prompt” — “`defaultProjectTrust: "always"` loads protected project resources”, while “`defaultProjectTrust: "ask"` or `"never"` skips them.”

Two documented limits: “Pi reads the project `sessionDir` setting while selecting or creating a session, before it resolves project trust,” and — the boundary that matters most — “Project trust does not limit what tool calls can access or affect. After Pi starts, enabled tools still use the operating-system permissions of the Pi process.” Context files load regardless: “Treat instructions in a folder as untrusted input even when you decline project trust.”

## Security posture

“Treat model-generated commands and code as untrusted. Pi can read, change, and execute files with the permissions of the account that started it … Extensions, package installers, language servers, and other child processes run with those same permissions unless an operating-system or virtualization boundary restricts them.” Injection is in scope: “Files, comments, instructions, command output, and model responses can steer the model through prompt injection.” The docs draw the line plainly: “Safety comes from limiting the files, credentials, processes, and network services Pi can access and affect if a generated action is wrong or hostile. Watching the transcript, using project trust, and reviewing changes do not create a security boundary.”

Extensions inherit everything: “An extension runs inside the Pi process with the same operating-system permissions. It can inspect prompts, tool calls, files, credentials, and session history, so load extensions only from sources you trust.”

## Isolation

| Method | Where Pi runs | What is isolated |
|---|---|---|
| Plain Docker | Container | “Pi, built-in tools, `!` commands, and extensions” |
| Docker Sandboxes | Managed sandbox | Same, with “Provider credentials remain on the host and are substituted by the proxy” |
| OpenShell | Local or remote sandbox | “Filesystem, process, network, and credential policies” |
| Gondolin extension | Host | “Built-in tools and `!` commands” in “a local micro-VM for tool execution while retaining the host interface” |

Caveats: “Mounting `~/.pi/agent` exposes your Pi credentials, settings, extensions, and sessions,” and “Tool-only isolation does not constrain the host Pi process or extension tools that do not use the isolated backend.” For unattended runs: “For untrusted or unattended work, use a container or another sandbox.”

## Diagnostics and reporting

`/debug` “writes the rendered terminal lines and current session messages to `pi-debug.log` in your agent directory. … It can contain prompts, model responses, tool output, file contents, and terminal data.” Security issues go through the repository Security Policy — “Do not open a public issue for a security-sensitive report” — with the scope note that “Expected local-agent behavior, prompt injection from untrusted content, lack of a built-in sandbox, and behavior from user-installed extensions or skills are generally outside the security boundary unless the report demonstrates a privilege-boundary bypass.”

## Official sources

- [Run Pi safely](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md) · [Command Line (Tools)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md) · [Settings (Tools)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/settings.md) · [Containerization](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/containerization.md) · [Extensions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md) · [Codemode](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/codemode.md)
