# Factory Droid corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| Factory Droid is only an interactive terminal coding agent. | ❌ FALSE | Current surfaces include CLI, Factory App, IDE/ACP integrations, headless execution, SDKs, Missions, Delegations, Automations, and hosted or remote compute. |
| `droid "<prompt>"` is the headless one-shot runner. | ❌ FALSE | It starts the interactive CLI with an initial prompt. Headless execution uses `droid exec`. |
| ACP support means every IDE exposes the same context and session features. | ❌ FALSE | VS Code-family, JetBrains, and Zed integrations have different context-sharing, diff, and session-restoration behavior. |
| JetBrains automatically supplies open files, selections, diagnostics, and native Droid diffs. | ❌ FALSE | The current JetBrains ACP integration does not automatically share those capabilities. |
| Zed restores past Factory Droid sessions in its Agent Panel. | ❌ FALSE | The current integration does not restore past Factory Droid sessions there. |
| All Factory settings use one universal precedence order. | ❌ FALSE | Hard controls and session defaults use opposite precedence systems. |
| Runtime overlays always outrank organization policy. | ❌ FALSE | Organization hard controls are authoritative; runtime overlays rank highly for session defaults but remain capped by hard controls. |
| Organization model defaults prevent users from choosing any other allowed model. | ❌ FALSE | A more-local session default can replace the org fallback, but `modelPolicy` still defines the allowed boundary. |
| `settings.local.json` is a separate top-level settings scope. | ❌ FALSE | It overlays `settings.json` at the same user or project level and inherits that level’s hierarchy position. |
| `.droid.yaml` is the current recommended project configuration file. | ❌ FALSE | It is legacy; use current `.factory/` settings, instructions, MCP, hooks, and skills. |
| A malformed system-managed organization file is silently ignored. | ❌ FALSE | A present malformed file produces an empty organization policy and logs the failure rather than silently bypassing it. |
| CLI sessions remain local unless a user explicitly opens Factory web. | ❌ FALSE | `cloudSessionSync` defaults to true and mirrors CLI sessions to Factory web unless disabled. |
| Personal BYOK models also apply to hosted web and mobile sessions. | ❌ FALSE | Personal custom models apply to CLI and desktop, not hosted web or mobile. |
| `apiKeyHelper` is a passive environment-variable reference. | ❌ FALSE | It executes a shell command to obtain credentials at request time. |
| A project can supply `apiKeyHelper` through folder or project settings. | ❌ FALSE | The helper is stripped from project and folder settings and honored in user and org-managed settings. |
| Only root `AGENTS.md` is supported. | ❌ FALSE | Droid searches applicable ancestor, context, personal, and dynamically discovered nested locations with documented compatible filenames. |
| Every nested instruction file is loaded at initial session startup. | ❌ FALSE | Nested instructions can be discovered dynamically after Droid reads files in a deeper directory tree. |
| `CLAUDE.md` is the only compatible alternative to `AGENTS.md`. | ❌ FALSE | The documented set also includes case variants of `AGENTS.md` and `Claude.md`; design guidance uses separate `DESIGN.md` names. |
| Spec Mode and High Autonomy are two names for the same control. | ❌ FALSE | Spec is a read-only planning interaction mode; Autonomy Level controls approvals for execution. |
| High Autonomy overrides an effective `ask` rule. | ❌ FALSE | An effective `ask` still prompts at High. |
| `--skip-permissions-unsafe` disables command blocks. | ❌ FALSE | It skips permission prompts but effective command blocks still apply. |
| Autonomy Level determines which tools are installed or available. | ❌ FALSE | It controls automatic approval; tool policy, MCP configuration, model support, and org controls separately restrict availability. |
| `droid exec` inherits an interactive session’s mutation default automatically. | ❌ FALSE | Headless execution is read-only by default and requires explicit autonomy for mutations. |
| A failed `droid exec` run rolls back earlier edits. | ❌ FALSE | Fail-fast permission errors leave earlier edits in the working tree. |
| Exit `0` from `droid exec` proves the task and tests succeeded. | ❌ FALSE | It means Droid finished; deterministic task and artifact validation is still required. |
| Native permission rules are wildcard command matchers. | ❌ FALSE | Prefixes are exact, ordered, case-sensitive tokens, and `*` is literal. |
| `['git', 'push']` matches `git -C /repo push`. | ❌ FALSE | The extra tokens change the ordered prefix, so that invocation does not match. |
| A successful `droid rules check` means the previewed command is allowed. | ❌ FALSE | Exit `0` means the check ran; the reported decision can still be `block`. |
| An organization `allow` cancels every lower-level `block`. | ❌ FALSE | After ID resolution, matching decisions rank `block > ask > allow`, including across different rule IDs. |
| Invalid local rules fail closed as block-all policy. | ❌ FALSE | Invalid local rules are skipped, which can expose a lower-priority valid rule with the same ID. |
| Plugins and dynamic configuration can author native permission rules. | ❌ FALSE | They cannot; eligible native sources are Org, Runtime, Folder, Project, and User, with trust requirements for project/folder rules. |
| Command permission rules inspect all behavior of an allowed program. | ❌ FALSE | They match command prefixes and do not inspect arbitrary post-launch behavior or provide OS isolation. |
| Sandbox write policy allows writes everywhere by default. | ❌ FALSE | Writes default to deny except the working directory and `allowWrite`; `denyWrite` wins. |
| Sandbox network access is unrestricted once a command is allowed. | ❌ FALSE | Network defaults deny except Factory and WorkOS domains plus configured allowed domains, with separate checks. |
| `per-command` confines the main Droid process and every child. | ❌ FALSE | That description applies to `whole-process`; `per-command` confines applicable commands individually. |
| If sandbox support is unavailable, Droid silently runs unsandboxed. | ❌ FALSE | The affected action is blocked, and whole-process Droid refuses to start without required isolation. |
| Factory sandboxing makes arbitrary untrusted code safe. | ❌ FALSE | Allowed writes and egress remain risk channels; hostile code still needs a container or VM and least-privilege credentials. |
| Custom droids share the parent’s context window. | ❌ FALSE | Each new Task invocation runs in a fresh context and returns a final message to the parent. |
| Custom droids can ask the user questions and spawn child droids. | ❌ FALSE | `AskUser` is disabled and `Task` is unavailable to subagents. |
| `tools: all` is the required way to give a custom droid every tool. | ❌ FALSE | The literal value is rejected; omit `tools` to allow all tools. |
| `mcpServers: []` inherits all parent MCP servers. | ❌ FALSE | It explicitly excludes all MCP servers; omit the field to inherit parent availability. |
| A subagent can exceed the organization autonomy ceiling when pinned to High. | ❌ FALSE | Resolved subagent autonomy is always clamped by the organization maximum. |
| A disallowed droid-pinned model runs anyway because it is explicit. | ❌ FALSE | Model validation falls back to the parent model rather than running the disallowed selection. |
| Skill discovery only checks direct children of `.factory/skills/`. | ❌ FALSE | Skill folders are searched recursively; the first directory containing `SKILL.md` becomes the skill directory. |
| Project skills always override organization plugin skills with the same name. | ❌ FALSE | Organization plugin skills outrank project, personal, built-in, mission, and automation skill sources. |
| `allowed-tools` enforces a skill sandbox. | ❌ FALSE | It is intended-tool metadata and neither grants tools nor restricts standard skill invocation. |
| `disable-model-invocation: true` disables direct user invocation too. | ❌ FALSE | It prevents automatic model selection while leaving direct user invocation available. |
| Markdown custom commands support `$1` and `$2`. | ❌ FALSE | Markdown commands use `$ARGUMENTS`; positional arguments apply to executable commands. |
| Executable custom commands run without the user’s environment. | ❌ FALSE | They inherit the environment and run from the current working directory. |
| Missions are just a different name for one custom-droid invocation. | ❌ FALSE | Missions use an orchestrator, workers, validation, progress tracking, and mission-specific configuration. |
| Automations are only cron jobs. | ❌ FALSE | They support scheduled, Slack, GitHub, and organization-enabled webhook triggers. |
| An automation’s trigger alone defines its privilege. | ❌ FALSE | Its instructions, run identity, repository/tool access, and compute determine the effective execution context too. |
| `${NAME}` expansion can rewrite any MCP command, argument, or URL. | ❌ FALSE | Expansion is limited to credential-bearing environment, header, and OAuth client fields. |
| MCP `enabledTools` overrides `disabledTools` when both name a tool. | ❌ FALSE | A tool present in both remains excluded. |
| MCP policy matches only the friendly configured server name. | ❌ FALSE | It matches remote hostname or stdio command/arguments. |
| Re-pointing an approved MCP server preserves the old persistent approval. | ❌ FALSE | Approval fingerprints cover the stdio command/arguments or remote URL; re-pointing invalidates the approval. |
| Project `mcp.json` is the recommended place for committed secrets. | ❌ FALSE | Project MCP files must not contain secrets; use supported credential references and local secret handling. |
| Every nonzero hook exit blocks the protected action. | ❌ FALSE | Exit `2` supplies blocking/corrective feedback where supported; other nonzero exits are non-blocking. |
| `SessionEnd` hooks can prevent termination. | ❌ FALSE | `SessionEnd` cannot block termination; documented stop prevention applies to `Stop` and `SubagentStop`. |
| Editing a hooks file immediately rewrites the active startup snapshot. | ❌ FALSE | Hooks are snapshotted at startup; external modifications produce a warning and should be reviewed through `/hooks`. |
| npm plugin installation allows lifecycle scripts to build the package. | ❌ FALSE | Installation uses `--ignore-scripts`; packages must ship ready-to-use files. |
| `--ignore-scripts` makes all installed plugin content safe. | ❌ FALSE | Packaged hooks, MCP servers, commands, and skill scripts can still execute or grant access. |
| Connectors run as locally configured MCP processes. | ❌ FALSE | Connectors use Factory’s managed remote connector service and do not run a local connector process. |
| Approving one connector action creates a persistent MCP permission. | ❌ FALSE | Connector approvals are action-specific and separate from persistent MCP permissions. |
| Connecting an app automatically installs its Delegation app. | ❌ FALSE | Connectors and Delegations are separate integration surfaces with separate setup. |
| SDK hooks are arbitrary in-process language callbacks. | ❌ FALSE | SDKs support skills and MCP tools, but hook definitions remain file-based. |
| Python persistent sessions use `session.run()`. | ❌ FALSE | There is no `session.run()`; each persistent turn uses `session.stream()`. |
| Missing or broken Python permission handlers fail open into approval. | ❌ FALSE | Absent, invalid, or raising handlers cause the request to be cancelled. |
| TypeScript SDK sessions close themselves after every streamed turn. | ❌ FALSE | `run()` cleans up automatically, but `createSession()` and `resumeSession()` sessions require explicit closure. |
| Shared delegations execute with the requester’s personal repository access. | ❌ FALSE | Shared work uses the configured service account and its repository access and connectors. |
| Slack and Google Chat DMs always use the shared service account. | ❌ FALSE | Those DMs use the sender’s personal identity, Git credentials, and connectors. |
| Personal delegation instructions replace the Apps agent-profile instructions. | ❌ FALSE | They are appended after the profile instructions and do not replace them or change shared execution identity. |
| Environment launches share one mutable working directory. | ❌ FALSE | Each launch creates an independent instance from the snapshot; changes do not flow to other instances or back to the baseline. |
| Rebuilding an Environment updates every persistent Computer created from it. | ❌ FALSE | A persistent Computer keeps its subsequent state and is not modified by later Environment rebuilds. |
| Remote Control requires opening inbound ports on the user-managed host. | ❌ FALSE | The host connects through Factory’s relay and requires no inbound ports. |
| Factory manages hardening and Git credentials on Remote Control hosts. | ❌ FALSE | The user manages the host, and Droid uses its existing Git configuration and credentials. |
| Factory worktrees isolate secrets and all external side effects. | ❌ FALSE | Worktrees separate working files; setup scripts, shared services, credentials, and `.worktreeinclude` copies still require independent review. |
