# Skills, MCP, and plugins

**Verified:** 2026-11-15

## Skills

Skill discovery merges project `.kimi-code/skills/` and `.agents/skills/`, user `$KIMI_CODE_HOME/skills/` and `~/.agents/skills/`, `extra_skill_dirs`, and built-in product skills, with precedence Project > User > Extra > Built-in — `merge_all_available_skills: true` combines everything instead of shadowing. Directory form requires frontmatter: “both name and description must be explicitly provided. Omitting either one will cause parsing to fail”; the flat form falls back to filename + first body line (≤240 chars). Types `prompt` / `inline` / `flow`; `whenToUse`, `disableModelInvocation`, and `arguments` tune triggering and parameters. Placeholders `$ARGUMENTS`, `$0`/`$1`, and `${KIMI_SKILL_DIR}` are substituted; “If the body contains no argument placeholders, text passed at invocation is appended to the end of the body” as `ARGUMENTS: <text>`. Invocation: `/skill:<name>`, the `/name` shorthand, dotted sub-skills, nesting up to three levels. The Skill tool can run inline skills only. `--skills-dir` “Replaces the automatically discovered user and project directories for this launch only.”

## MCP servers

Servers are declared in `~/.kimi-code/mcp.json` (user) and `.kimi-code/mcp.json` (project, wins per-server, trust-gated). Transports: stdio (`command`, `args`, `env`, optional `cwd`, `environment_id` — “environment_id starts it inside a remote environment instead”), HTTP, and legacy SSE (`"transport": "sse"`). Other fields: `bearerTokenEnvVar`, `headers`, `enabled`, `deferred`, `startupTimeoutMs` (default 30000), `toolTimeoutMs`, `enabledTools` / `disabledTools` with `*` and `**` wildcards. Tools surface as `mcp__<server>__<tool>`. Attachment-based servers can return `kimi-file://` references, and “structuredContent” from tool results is preserved. Approvals reuse the standard flow — “Approve for this session” in the approval dialog automatically allows subsequent calls of the same kind within the current session — and “In Ask When Needed mode, MCP tool calls are automatically approved.”

Deferred loading keeps tool definitions out of the top-level list: the model sees “a manifest of loadable tools, loads full definitions on demand through the built-in select_tools tool, and can call them in the same turn once loaded.” The table flags it “Experimental: set to true to let the model load this server's tools on demand,” with per-server `deferred` beating global `[mcp].default_deferred`; the value is captured at connect time. Reconnection backs off 5000 ms → 300000 ms for workspace-level servers, but “Servers that cannot recover on their own — an unknown command, an invalid configuration, or static credentials the server rejects — are not retried and stay failed, and session-scoped ephemeral servers are not retried either.”

Trust is the gate for project servers: “When Kimi Code finds project-level MCP servers in an untrusted folder, it shows each server's transport and launch target in the workspace trust prompt,” and “Headless runs (for example kimi -p in CI) cannot show the trust prompt, so project-level MCP servers stay disabled there unless the workspace is already trusted.” The `/mcp` command inspects servers live.

## Plugins

Plugins bundle slash commands, skills, agents, system-prompt contributions, and hook definitions behind one manifest. “Plugins are currently installed per-user and apply to all projects; project-level installation scope is not yet supported.” Installation uses the plugin manager under `$KIMI_CODE_HOME/plugins/installed.json` + `plugins/managed/`. Prompt contributions are budget-capped twice: “Each field (the inline `systemPrompt` and the `systemPromptPath` file) is limited to 32 KB (UTF-8 bytes): oversized content is ignored and reported in the plugin diagnostics,” and “one prompt build injects at most 64 KB of instructions; contributions beyond the budget are skipped with a warning, including a single plugin whose inline text and file together exceed that budget.” Plugin hooks can be blocking like user hooks but always fail open.

Official plugins include “Kimi Computer Use” (“lets AI operate your desktop apps directly, clicking, dragging, scrolling, and typing”), the Kimi Browser Extension (WebBridge), and Datasource; they surface in `/plugins` and can be toggled per session. Plugin-provided skills and agents participate in the standard precedence chains (Plugin tier, below User).

## Official sources

- [Skills](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/skills.html) · [MCP](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/mcp.html) · [Plugins](https://www.kimi.com/code/docs/en/kimi-code-cli/plugins.html) · [Tools reference](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/tools.html)
