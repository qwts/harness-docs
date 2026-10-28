# GitHub Copilot corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| `COPILOT_HOME` changes only the trusted-folder file location. | ❌ FALSE | It relocates the CLI configuration home, affecting user-level configuration locations more broadly. |
| Trusted folders and project MCP trust are unrelated. | ❌ FALSE | Project MCP files are skipped in untrusted directories. |
| `--allow-all-tools` overrides an explicit deny. | ❌ FALSE | Deny rules and enterprise restrictions still apply. |
| `--allow-all` only suppresses tool prompts. | ❌ FALSE | It combines tool, path, and URL bypasses. |
| Default path access is the whole filesystem. | ❌ FALSE | Default access is the working directory, descendants, and system temp directory. |
| CLI path and URL checks are a complete shell-security boundary. | ❌ FALSE | Shell extraction is heuristic and has documented blind spots. |
| Local CLI sandboxing is enabled by default. | ❌ FALSE | It is off by default; local and cloud sandboxes are separate. |
| Copilot CLI reads `.vscode/mcp.json`. | ❌ FALSE | Use `.mcp.json`, `.github/mcp.json`, or user MCP configuration as documented. |
| `.github/mcp.json` always beats `.mcp.json`. | ❌ FALSE | In one directory, `.mcp.json` wins; closer project files also win by directory proximity. |
| Cloud agent reads the CLI's project MCP files as its repository settings. | ❌ FALSE | Cloud repository MCP is configured on GitHub and is a separate surface. |
| Cloud agent supports MCP resources and prompts as well as tools. | ❌ FALSE | Current cloud-agent/code-review documentation supports tools only. |
| Cloud agent asks before every configured MCP call. | ❌ FALSE | Configured MCP tools may be invoked autonomously. |
| Applicable instruction files have a general documented precedence order. | ❌ FALSE | They are combined; avoid conflicts. |
| All custom agents are deduplicated by their display `name`. | ❌ FALSE | CLI IDs derive from filenames; the display name is not the deduplication key. |
| Omitting `tools` disables all tools. | ❌ FALSE | Omitted or `["*"]` enables all available tools; `[]` disables all. |
| Custom subagents inherit repository instructions automatically. | ❌ FALSE | Set `include-custom-instructions: true`; directly selected session agents already receive them. |
| Any Markdown file under a skills directory is a valid skill manifest. | ❌ FALSE | The manifest must be named `SKILL.md` and include required frontmatter. |
| Agent Plugins and legacy Copilot plugins have identical component-path rules. | ❌ FALSE | Agent Plugins use fixed portable paths plus namespaced client components; legacy manifests can configure paths. |
| Every Copilot IDE has full CLI customization parity. | ❓ UNVERIFIED | Support varies by IDE and feature; use the current feature matrix rather than inferring parity. |
