# DeepSeek corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| The registry’s DeepSeek harness is just the DeepSeek model API. | ❌ FALSE | It maps to DeepSeek Harness (`dsh`), an open-source plugin-composed agent runtime. |
| DeepSeek Harness is stable production software. | ❌ FALSE | It is experimental developer-preview software, expects breaking changes, and has not undergone a security audit. |
| `dsh` is primarily an interactive terminal UI. | ❌ FALSE | It is a profile launcher; Web/Desktop are interactive surfaces, while headless, ACP, and SDK serve automation. |
| The `dsh tui` help example proves a TUI ships by default. | ❌ FALSE | The example explicitly assumes a terminal profile is installed. |
| ACP and SDK have separate public Node binaries. | ❌ FALSE | They are profiles launched through `dsh`. |
| `$DSH_HOME` defaults to `~/.deepseek`. | ❌ FALSE | It defaults to `~/.dsh`. |
| Profile patches override the home-wide patch. | ❌ FALSE | `$DSH_HOME/cordis.patch.yml` applies after the profile layer and outranks it. |
| Profile configuration deep-merges every `config` key. | ❌ FALSE | A patch replaces the targeted row’s complete `config` value. |
| Every plugin/package change hot-reloads immediately. | ❌ FALSE | Bundle membership changes require restart; supported profile/home patch edits can use HMR. |
| API keys are readable back from the Models page after saving. | ❌ FALSE | Saved keys are write-only; the page receives redacted descriptors. |
| The managed credential file is copied into `process.env`. | ❌ FALSE | It is resolved as a credential store without being materialized into the process environment. |
| Changing the selected model rewrites an existing session’s model. | ❌ FALSE | Selection changes the default for new sessions; an existing session retains its logged model. |
| A local Web/Desktop UI guarantees session data remains local. | ❌ FALSE | DeepSeek session-log contribution is enabled by default and can export log data with subsequent DeepSeek requests. |
| Disabling OTel also disables DeepSeek session-log upload. | ❌ FALSE | They are separate paths with separate controls. |
| All profiles default to `workspace-write`. | ❌ FALSE | Base-backed profiles do; standalone `sdk-minimal` pins `danger-full-access`. |
| Full Access keeps interactive approval prompts enabled. | ❌ FALSE | The preset bundles `danger-full-access` sandbox with approval `never`. |
| Changing Web permission defaults updates an open session. | ❌ FALSE | Stored defaults affect later sessions; current/resumed sessions retain effective permissions. |
| `workspace-write` blocks reads outside the workspace. | ❌ FALSE | It restricts file mutations; reads are not confined by that mode. |
| `workspace-write` also provides network egress control. | ❌ FALSE | Network is outside the file-effect policy; enabled fetch calls can reach loopback/private hosts without per-call confirmation. |
| The sandbox plugin’s own fallback is full access. | ❌ FALSE | Its fail-safe configuration default is `read-only`; deployments explicitly opt into broader modes. |
| Additional writable roots may be symlink aliases. | ❌ FALSE | Grants require physical absolute directories; symlinked roots are rejected. |
| Granting shared `.git` metadata only permits commit objects to change. | ❌ FALSE | It can also permit modification of hooks and configuration later executed by host Git operations. |
| Only root `AGENTS.md` is supported. | ❌ FALSE | Global chains and project `AGENTS.md`/`CLAUDE.md` plus additive local overlays are supported. |
| Instruction discovery watches every edit continuously. | ❌ FALSE | Refresh is driven by structured filesystem touches and resume, with working-directory reconciliation. |
| Shell `cd` triggers discovery of nested instructions. | ❌ FALSE | Shell navigation is not a structured filesystem discovery event. |
| Instruction content is model-summarized when over budget. | ❌ FALSE | Broader files are omitted and the most specific file can be truncated; no model compression is used. |
| Instruction symlinks are confined to the repository. | ❌ FALSE | They are followed, including off-tree targets. |
| `sdk-minimal` loads the same local instructions as base profiles. | ❌ FALSE | It omits local instruction discovery. |
| Any nested `SKILL.md` is discovered recursively. | ❌ FALSE | Only direct `<root>/<name>/SKILL.md` bundles and top-level `<root>/<name>.md` files are recognized. |
| Invalid skill invocation metadata fails open into the catalog. | ❌ FALSE | The skill is dropped with a warning. |
| Mounting `dsh-subagent` alone enables delegation. | ❌ FALSE | A provider and model-facing delegation tool are also required. |
| Every external subagent supports durable follow-up messages. | ❌ FALSE | ACP, DSH SDK, Codex, and Claude Code external children execute once; local spawn/fork children are continuable. |
| Child agents can widen their delegated permission scope. | ❌ FALSE | In-process child scope is fixed at creation and approval-required operations are rejected automatically. |
| Agent Teams are the stable default name for subagents. | ❌ FALSE | Agent Teams are a separate optional experimental capability. |
| MCP servers are enabled by default. | ❌ FALSE | No server is enabled by default; each server requires a configured client row. |
| MCP tool names use the remote server’s advertised name. | ❌ FALSE | Names use the locally configured `serverName`: `mcp__<serverName>__<tool>`. |
| An MCP startup failure always aborts the harness. | ❌ FALSE | `failOnStartupError` defaults to false, so the harness can start without that server’s tools. |
| Stdio MCP servers inherit every ambient credential variable. | ❌ FALSE | Ambient credential-like and `DSH_*` names are scrubbed before explicit `env` entries are merged. |
| Claude Code and Codex hooks are stable, fully compatible built-ins. | ❌ FALSE | They are experimental compatibility bridges with documented event, field, and lifecycle limitations. |
| Any hook process failure blocks the protected action. | ❌ FALSE | Most failures are non-blocking; only supported blocking outcomes such as exit 2 stop an action. |
| `continue: false` reliably terminates the whole run. | ❌ FALSE | It is recorded but has no run-level halt effect in the shared hook protocol. |
| A plugin listed under `dsh-plugin` is security-reviewed. | ❌ FALSE | The topic aids discovery; plugins remain executable code requiring review. |
| Headless always exits 0 when it prints an answer. | ❌ FALSE | Exit 0 requires final turn reason `completed`; other outcomes exit 1. |
| Headless JSON mode emits one final JSON envelope. | ❌ FALSE | It emits newline-delimited run events. |
| `--public-url` authorizes a public proxy hostname. | ❌ FALSE | It only advertises a URL; `--trusted-host` separately admits the authority. |
| Desktop closing the window stops all running tasks. | ❌ FALSE | Ordinary close hides the window while the Host and tasks continue; quitting is separate. |
| Python SDK execution requires system Node.js. | ❌ FALSE | The wheel includes a matching native runtime and `dsh`; normal SDK execution needs no system Node. |
| `sdk-minimal` is a safe reduced-permission profile. | ❌ FALSE | It pins full access, has no approval service, and its shell can modify any runtime-visible path. |
| Adding the minimal editor adds workspace confinement. | ❌ FALSE | The local filesystem provider does not confine access to the runtime working directory. |
