# Safety, approvals, and permissions

**Verified:** 2026-11-16 (Vibe Code CLI ≥ 2.25.4; docs site main)

## Layered controls

“Vibe Code uses **layered controls**”; “**Agents** are configuration overrides … They define what gets auto-approved”; “**Trusted folders** define whether Vibe can load project-level configuration, skills, and agent files from the current directory”; “**Per-tool permissions** let you tighten or loosen access for specific tools.”

Two extra guards: “Vibe also asks for confirmation whenever a tool tries to read, write, or run a command **outside the working directory** and the directories added with `--add-dir`, unless every tool call is auto-approved,” and “It also asks before a tool reads or edits `.env` and `.envrc` files, even when the tool's permission is `always`. A path listed in the tool's `allowlist` skips both checks.”

## Agents and approval behavior

| Agent | Behavior |
|---|---|
| `ask` | “Asks before running any tool.” |
| `accept-edits` | “**Default.** Auto-approves file edits. Still asks for shell commands and other sensitive tools.” |
| `smart-approve` | “Classifies each tool call. Runs the calls it judges safe and asks you before risky ones.” |
| `auto-approve` | “Auto-approves every tool execution, including shell commands”; “Your administrator can disable it.” |

Programmatic mode: “Nobody can answer an approval prompt, so Vibe denies every tool call that needs approval. Pass `--auto-approve` (or `--yolo`) to approve every tool call, only in a sandboxed environment. Vibe ignores these flags when your administrator sets `enable_auto_approve = false`.”

## Smart approve

For each call needing approval, in order: “1. **Applies your permission rules first.** … 2. **Reuses approvals you already gave** for the same call in the current session. 3. **Classifies the call.** A model judges the call from your request and the tool arguments. … 4. **Holds back risky calls.** Vibe sends the reason back to the agent so that it can try a safer approach. Vibe asks you to confirm when the agent repeatedly makes risky calls, retries the same call, or when the classifier can't reach a verdict.” “A call flagged as risky never runs without your confirmation. In programmatic mode, no one can confirm, so Vibe denies risky calls.”

The warning: “Smart approve is a classifier, not a sandbox … If you work with untrusted code or sensitive credentials, use the `ask` agent or an isolated environment.” Classifier model selection: `[utility_models].smart_approve` → “a small, fast Mistral model, when your Mistral provider serves one” → “your session's model, which can cost more per call”; a non-matching alias “shows a warning and falls back to automatic selection.” Rollout note: “Smart approve is rolling out gradually.”

## Trusted folders

“A project can ship its own Vibe configuration … in a `.vibe/` directory at the repository root. Because that configuration **changes how Vibe behaves**, Vibe will only load it from directories **you have explicitly trusted**.” Trustable content: “A `.vibe/` directory with a `config.toml` file or an `agents/`, `skills/`, `prompts/`, `plugins/`, or `tools/` subdirectory”; “An `.agents/skills/` directory (Agent Skills standard skills)”; “An `AGENTS.md` file (project-level agent instructions).” The dialog offers “**Trust folder**, **Trust full repo** (offered when you start Vibe in a subdirectory of a git repository), or **Don't trust**” — “The dialog doesn't analyze these files, so review them yourself.” Decisions persist in `~/.vibe/trusted_folders.toml`; untrusted means “project-level configuration is ignored and Vibe prints a warning. User-level configuration in `~/.vibe/` still applies.”

Temporary trust: “The `vibe --trust` flag trusts the working directory for the current invocation only and does not persist … **Programmatic mode never prompts for trust.**” “`--worktree` also trusts its worktree for the run. Vibe loads project configuration from directories added with `--add-dir` without asking for trust, so only add directories you trust.”

## Per-tool permissions

`[tools.<name>]` with `permission = "always" | "ask" | "never"` applies to “built-in tools (`read_file`, `write_file`, `edit`, `bash`, etc.) as well as MCP tools and connector tools.” Toggling: `enabled_tools` / `disabled_tools` accept “exact names, globs, or regex patterns”; `disabled_tools` is “applied after `enabled_tools`”, whereas `disabled_skills` is “applied when `enabled_skills` is empty”. **Bash command lists**: “Some read-only commands (`ls`, `pwd`, `cat`, `git status`, etc.) are auto-allowed by default. Set `allowlist` and `denylist` to choose the commands Vibe always runs or always blocks. Each setting replaces its default list.”

**Outside the working directory**: `outside_workdir = "ask"` (default) or `"deny"` — “the agent gets an error and keeps working” — settable globally or per tool. Boundary caveat: “Shell tools only check paths written in the command, not paths a script builds, so keep a shell tool that must stay inside the working directory on `ask`.” **MCP tools** are addressed “as `{server_name}_{tool_name}`” (e.g. `[tools.fetch_server_get]`).

## Approval UX per surface

“The approval model is consistent across the CLI, the VS Code extension, and Vibe Code Web.” CLI keys: `1`/`y` — “Allow the action once”; `2` — “Allow it for the rest of the session”; `3` — “Always allow it”; `4`/`n`/`Escape` — “Deny the action.” VS Code: `Mod+Y` approve, `Mod+Alt+Y` alternate, `Mod+Shift+Y` broader permission, `Mod+Alt+N` reject (“`Mod` means `Command` on macOS and `Ctrl` on Windows or Linux”); “A dedicated UI for per-tool permissions is still coming soon; in the meantime, configure them through the same `config.toml` files used by the CLI.” Web: “Approvals appear in the browser UI, and repository-level actions … follow the GitHub permissions you granted.”

Best practices from the docs: “Start in `ask` or a plan agent for unfamiliar code”; “Commit before risky runs.”

## Security advisory (fixed)

“Mistral Vibe **2.25.4**, released on **12 September 2026**, includes fixes for six reported vulnerabilities in shell permission checks (CVE-2026-87983 through CVE-2026-87988).” Before the fix, “Vibe could execute specially crafted shell commands without the expected approval prompt because the permission checks did not account for all command syntax, options or file paths,” letting “malicious instructions in content processed by the agent” read files outside the workspace, write there, or “execute code with the permissions of the user running Vibe.” “Versions before **2.25.4** may be affected.” Run at least 2.25.4.

## Official sources

- [Safety, approvals, and permissions](https://docs.mistral.ai/vibe/code/safety-approvals-permissions) · [Admin config](https://docs.mistral.ai/vibe/code/cli/admin-config) · [Advisory MAI-2026-003](https://docs.mistral.ai/resources/security-advisories/MAI-2026-003) · [Hooks](https://docs.mistral.ai/vibe/code/cli/hooks)
