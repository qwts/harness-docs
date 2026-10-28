# CLI, IDE, cloud, and SDK surfaces

**Verified:** 2026-10-29

## Surface capability matrix

Shared configuration support does not imply feature parity.

| Capability or surface | VS Code / JetBrains | Desktop | CLI | SDK / Kanban |
|---|---:|---:|---:|---:|
| Rules and shared configuration | Yes | Yes | Yes | Surface-dependent |
| Subagents | No | Yes | Yes | Not established here |
| Plugins | No | Not established here | Yes | Yes |
| Agent teams | No | Not established here | Yes | Yes |
| File hooks | Yes, with extension layout | Not established here | Yes, with CLI layout | Use plugin hooks where applicable |
| ACP | Not applicable as a CLI flag | Not established here | Yes | Integration-dependent |

“Not established here” is not a claim of absence. It marks a parity claim that this increment does not make. Consult the feature’s current official page before relying on it for another host.

## CLI basics

Install the documented package globally:

```sh
npm i -g cline
```

Common invocations:

```sh
cline                         # interactive session
cline "inspect this repo"    # one-shot prompt run
cline -p                     # Plan mode; --plan is equivalent
cline --json "summarize"     # JSON event output
cline --acp                  # Agent Client Protocol mode
cline --worktree "task"      # detached worktree execution
```

A prompt run defaults to Act mode with auto-approval enabled. In ACP mode, `--auto-approve` defaults to `false`. Check effective options instead of applying one default to every invocation.

`--worktree` creates a detached worktree under `~/.cline/worktrees/`. A separate worktree reduces working-tree interference; it is not, by itself, a credential or operating-system security sandbox.

## Headless and JSON operation

Headless behavior is triggered by any of the following:

- `--json`;
- piped stdin;
- redirected stdout.

JSON agent runs emit **newline-delimited JSON events**, not one final JSON envelope. To extract the final answer from the documented `run_result` event:

```sh
cline --json "summarize this changelog" | jq -r 'select(.type == "run_result") | .text'
```

Errors, including timeout errors, are emitted to stderr as JSON error objects. A consumer should parse the event stream and stderr rather than scrape terminal formatting.

The default timeout is `0`, meaning no timeout. Set an explicit timeout and external job limits for unattended automation. A process exit or final response is not a substitute for tests and task-specific validation.

## ACP

ACP is an integration mode, not proof that the connected client and standalone CLI share every default. In particular, auto-approval defaults to false in ACP mode, unlike an ordinary prompt run. Approval UX and policy can also be mediated by the client.

## Cloud and Kanban boundaries

This increment makes only the explicitly verified Kanban claims: plugins and agent teams are documented for SDK, CLI, and Kanban. It does not infer that a cloud or Kanban run has the same local filesystem layout, interactive approval UI, worktree behavior, or extension hooks as a workstation CLI.

For a cloud deployment, verify its current environment, persistence, secrets handling, and execution policy separately. Local paths such as `~/.cline/data/db/teams.db` describe documented local state and should not be assumed to be a portable cloud database architecture.

## SDK packages

The main package is `@cline/sdk`, requiring Node.js 22 or later.

| Package | Role |
|---|---|
| `@cline/core` | Full harness: built-in tools, sessions, approvals, persistence, hub/remote backends, scheduling, plugins, and teams |
| `@cline/agents` | Lighter stateless agent loop; does not include built-in tools by default |

Choosing `@cline/agents` does not implicitly reproduce the full ClineCore tool and session environment. Applications must deliberately supply the capabilities they need.

Current built-in ClineCore tool names include:

```text
read_files
search_codebase
run_commands
fetch_web_content
apply_patch
editor
skills
ask_question
submit_and_exit
```

Do not present older XML-style tool names as the current SDK/ClineCore tool contract. Hook policies should target current names and be reviewed when upgrading.

## Sources

- [CLI overview](https://docs.cline.bot/usage/cli-overview)
- [CLI reference](https://docs.cline.bot/cli/cli-reference)
- [Configuration](https://docs.cline.bot/getting-started/config)
- [Subagents](https://docs.cline.bot/features/subagents)
- [Agent teams](https://docs.cline.bot/cli/agent-teams)
- [Plugins](https://docs.cline.bot/customization/plugins)
- [SDK overview](https://docs.cline.bot/sdk/overview)
- [ClineCore](https://docs.cline.bot/sdk/clinecore)
- [Tool reference](https://docs.cline.bot/tools-reference/all-cline-tools)
