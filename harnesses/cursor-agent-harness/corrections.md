# Cursor corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| Any `.md` file under `.cursor/rules/` is a project rule. | ❌ FALSE | Project rules use `.mdc`; plain `.md` there is ignored. |
| Only the single highest-precedence Cursor rule is loaded. | ❌ FALSE | Applicable rules are merged; precedence matters for conflicts. |
| `AGENTS.md` is unsupported by Cursor. | ❌ FALSE | Cursor supports it as a simpler Markdown instruction format, including nested behavior. |
| Workspace permission arrays replace user permission arrays. | ❌ FALSE | User and workspace arrays are concatenated. |
| IDE settings override administrator policy. | ❌ FALSE | Team admin > `permissions.json` > IDE settings. |
| Auto-review guarantees that every command is sandboxed. | ❌ FALSE | It sandboxes where possible and uses best-effort automatic review for remaining eligible calls. |
| Run Everything is equivalent to Auto-review. | ❌ FALSE | Run Everything bypasses ordinary command-by-command review. |
| Cloud subagents inherit the parent's local MCP configuration. | ❌ FALSE | Cloud subagents use team/cloud MCP configuration. |
| Every locally discovered skill is synced to Cloud Agents. | ❌ FALSE | Personal sync uses `~/.cursor/skills/`; automation runs do not load synced personal skills. |
| `SKILL.md` needs only a Markdown body. | ❌ FALSE | `name` and `description` frontmatter are required. |
| Cursor MCP supports tools only. | ❌ FALSE | Cursor documents tools, prompts, resources, roots, elicitation, and Apps. |
| Cloud hooks run during every early exploratory turn. | ❌ FALSE | They do not run during early read-only exploratory turns. |
| Cloud `preToolUse.updated_input` rewrites backend-hosted HTTP MCP calls. | ❌ FALSE | It can deny but ignores `updated_input` for those calls. |
| Cloud `postToolUse` always receives unredacted HTTP MCP output. | ❌ FALSE | Output is redacted for backend-hosted HTTP MCP tools. |
| Agent Plugins and Cursor Plugins have the same manifest path. | ❌ FALSE | Agent Plugins use root `plugin.json`; Cursor Plugins use `.cursor-plugin/plugin.json`. |
| Cursor expands portable `${PLUGIN_ROOT}` and `${PLUGIN_DATA}` in plugin `mcp.json`. | ❌ FALSE | Use `${CURSOR_PLUGIN_ROOT}` for the plugin root; the portable placeholders are not expanded there. |
| Cursor CLI is invoked as `cursor-agent`. | ❌ FALSE | The documented executable is `agent`. |
| Repository `.cursor/environment.json` always wins over saved environments. | ❌ FALSE | Personal and team saved environments have higher priority. |
| Self-Hosted Machines read `.cursor/environment.json`. | ❌ FALSE | They do not. |
| Cloud Builds preserve running services and exported environment variables. | ❌ FALSE | Builds preserve disk state only. |
| A failed Build immediately replaces the active environment. | ❌ FALSE | The last successful active Build remains in use. |
