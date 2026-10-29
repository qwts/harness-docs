# CLI, app, cloud, and SDK surfaces

**Verified:** 2026-10-29

## Surface model

Factory surfaces share parts of the Droid harness but do not expose identical context, persistence, identity, or approval behavior.

| Surface | Primary execution model | Key distinction |
|---|---|---|
| Droid CLI | Interactive local session | Uses configured session defaults and terminal-based controls |
| Factory App | Visual local or connected-machine workspace | Adds review, preview, project/machine selection, and managed worktrees |
| IDE integration | CLI extension or ACP front end | Editor context and session restoration differ by IDE |
| `droid exec` | Non-interactive one-shot or protocol-driven subprocess | Read-only by default; mutation requires explicit autonomy |
| Python/TypeScript SDK | Application-owned local Droid process or daemon client | Caller handles streaming, interactions, lifecycle, and validation |
| Delegations | Requests from collaboration/tracker apps | Shared service-account execution differs from personal DM execution |
| Automations | Triggered Droid sessions | Trigger, instructions, run identity, and compute form one job |
| Environments | Fresh instances from reusable snapshots | Instances do not share working-tree changes or update the baseline |
| Droid Computers | Persistent Factory-managed compute | State can survive across sessions |
| Remote Control | User-managed computer through Factory relay | Uses the host’s existing development setup and Git credentials |

## Interactive CLI and Factory App

`droid` starts the interactive terminal surface. `droid "<prompt>"` starts the same interactive surface with an initial prompt rather than invoking the headless runner. Normal, Spec, Mission, and Autonomy controls apply across CLI and Factory App sessions, subject to surface and organization availability.

Factory App provides a visual workspace for sessions, diffs, previews, project switching, model controls, and machine connections. Local, web, and mobile review surfaces can connect to the same Factory runtime; that does not make their execution hosts or persistence identical.

### Factory-managed worktrees

Factory App can create separate Git worktrees on a local machine or connected Droid Computer. Worktrees isolate working files, not repository history, credentials, external services, or every side effect.

Shared setup profiles live in:

```text
.factory/worktree-setups/*.yaml
```

They can run setup and cleanup scripts and provide an initial prompt. `.worktreeinclude` can copy Git-ignored files into new worktrees before setup. Those files are ignored by Git precisely because they often contain local configuration or credentials, so every included path requires secret review. Worktree separation is not a secret-isolation boundary.

For `droid exec` worktrees, clean exec-created worktrees are automatically removed; dirty worktrees are preserved. Preserve and inspect changes rather than assuming all temporary checkouts are deleted.

## IDE and ACP differences

| Editor surface | Documented behavior |
|---|---|
| VS Code, Cursor, Windsurf | Shares active file, selection, open files, diagnostics, and IDE-native diff/file actions |
| JetBrains IDEs | Uses ACP with model and autonomy controls; does not automatically share open files, selections, diagnostics, or native Droid diffs |
| Zed | Uses ACP, supports `@`-tagged file context and Zed MCP context servers; does not restore past Factory Droid sessions from its Agent Panel |
| Other editor terminals | Ordinary CLI behavior; reference files by path or provide context manually |

ACP is a protocol, not a guarantee of feature parity across clients. In particular, do not infer VS Code-family context sharing or CLI session restoration from the presence of an ACP integration.

## Droid Exec

`droid exec` is Factory’s headless execution mode:

```sh
droid exec "analyze code quality"
droid exec --auto low "add JSDoc comments"
droid exec --auto medium "install deps, run tests, and fix failing checks"
droid exec --session-id <session-id> "continue"
```

Each ordinary run is a non-interactive pass that writes output and exits. The default is read-only. `--auto low`, `--auto medium`, and `--auto high` explicitly enable progressively broader mutation and command behavior, still subject to command policy, sandbox rules, tool availability, and organization controls.

An effective `ask` cannot collect an interactive approval in a one-shot run. If Droid attempts an action above the current autonomy level, execution fails fast and exits nonzero. **Edits made before that failure remain in the working tree.** Check and clean the workspace before retrying with broader permissions.

Output formats are:

```text
text
json
stream-json
stream-jsonrpc
acp
```

`json` produces one final result object; `stream-json` emits newline-delimited progress events. Exit `0` means Droid finished, not that tests passed, a deployment succeeded, or the requested result is correct. Validate artifacts and external effects deterministically.

### Raw JSON-RPC

A long-lived client can drive multi-turn execution over stdio:

```sh
droid exec --input-format stream-jsonrpc \
  --output-format stream-jsonrpc --auto low
```

Core protocol methods and events include:

```text
droid.initialize_session
droid.load_session
droid.add_user_message
droid.session_notification
droid.request_permission
droid.ask_user
```

The client owns request IDs, response matching, streaming, permission/question responses, timeouts, process cleanup, and session persistence. In this mode, `disableBuiltinSkills` is supplied through initialization or load requests rather than the CLI flag. `--disable-builtin-skills` preserves non-built-in skill sources but is unsupported in ACP mode.

## Python SDK

Install with Python 3.10 or newer:

```sh
pip install droid-sdk
```

The SDK requires `droid` on `PATH` and either CLI authentication or `FACTORY_API_KEY`. It runs a local Droid subprocess.

| Goal | API |
|---|---|
| One prompt | `await run(...)` |
| Persistent turns | `Session(...)` |
| Resume saved context | `Session.resume(...)` |
| Discover models | `await list_models(...)` |
| Find sessions | `await list_sessions(...)` |

There is no `session.run()`; persistent turns use `session.stream()`. `run()` cleans up resources it creates, while manually managed sessions must be closed through their context manager or caller-owned lifecycle.

Permission and question requests require valid handlers. Python cancels a request when the handler is absent, invalid, or raises; it does not silently convert a broken handler into approval. A turn’s normal outcome is represented as a result value, while setup, transport, protocol, timeout, and cancellation failures can raise exceptions.

## TypeScript SDK

Install with Node.js 18 or newer:

```sh
npm install @factory/droid-sdk
```

Import Node APIs from `@factory/droid-sdk/node`. The SDK can start Droid as a Node subprocess or connect to an existing daemon.

| Goal | API |
|---|---|
| One prompt | `run()` |
| Persistent turns | `createSession()` |
| Resume saved context | `resumeSession()` |
| Discover models | `listModels()` |
| Find sessions | `listSessions()` |

`run()` closes its own session. Sessions returned by `createSession()` and `resumeSession()` must be closed explicitly, preferably in `finally`. Each session accepts one active turn at a time; use separate sessions for parallel work.

Both SDKs support skills, external MCP, and in-process SDK MCP tools. Hook definitions remain file-based rather than becoming arbitrary in-process SDK hook callbacks. Applications must still implement permission and question handling, tool restrictions, result checking, and resource cleanup appropriate to their threat model.

## Delegations and execution identity

Delegations accept work from Slack, Linear, Jira, and Microsoft Teams; Google Chat is documented as in development. Shared delegations run on remote compute using the organization’s Apps agent profile.

| Request type | Execution identity and access |
|---|---|
| Shared channel, space, or issue delegation | Configured service account, with that account’s repository access and connectors |
| Slack or Google Chat direct message | Sender’s personal identity, Git credentials, and connectors |

The selected shared service account needs access to every repository in the agent profile’s Environment, not merely the repository named in one request. Personal delegation instructions are appended after Apps agent-profile instructions; they do not replace those instructions or change shared execution identity. Personal model and instruction defaults likewise do not turn a shared run into a personal-identity run.

Review the service account, repository permissions, connectors, plugins, compute, and source-control review rules as one system. Delegation from a chat app is an entry point, not an approval or identity boundary.

## Environments, Computers, and Remote Control

### Environments

An Environment is a reusable development-environment snapshot containing selected repositories and setup results. Every launch creates an independent instance. Changes in one instance do not flow back into the snapshot or into another instance; rebuild the Environment to change the baseline and push or otherwise preserve durable work.

Environment-backed Delegations, Automations, and Mission workers can therefore start from the same baseline without sharing working-tree changes. Compute limits, repository access, provider capacity, and model allowances still apply.

### Droid Computers

A Droid Computer is persistent Factory-managed compute for work that needs files and configuration to survive across sessions. Creating one from an Environment copies the baseline at creation; later Environment rebuilds do not modify the existing Computer.

### Remote Control

Remote Control connects a user-managed Linux, macOS, or Windows computer through Factory’s relay:

```sh
droid daemon --remote-control
```

No inbound ports need to be opened. Factory does not provision or maintain that computer, and Droid uses its existing development setup and Git credentials. The owner remains responsible for host hardening, updates, credential scope, and persistence. Relay connectivity is not host isolation.

## Sources

- [Droid CLI reference](https://docs.factory.com/droid-cli/cli-reference)
- [Factory App](https://docs.factory.com/factory-app/overview)
- [Factory App worktrees](https://docs.factory.com/factory-app/worktrees)
- [IDE integrations](https://docs.factory.com/ide-integrations)
- [Droid Exec](https://docs.factory.com/droid-exec/overview)
- [Python SDK](https://docs.factory.com/sdk/python)
- [TypeScript SDK](https://docs.factory.com/sdk/typescript)
- [Delegations](https://docs.factory.com/delegations)
- [Environments](https://docs.factory.com/environments/overview)
- [Run Droids on an Environment](https://docs.factory.com/environments/run-droids)
- [Automations](https://docs.factory.com/automations/overview)
- [Remote Control](https://docs.factory.com/remote-control)
