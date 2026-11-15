# Agents, subagents, and Tower

**Verified:** 2026-11-15

## Built-in subagents

Three subagents ship built in: `coder` (the default delegation target, full toolset), `explore` (read-only research), and `plan` (no shell). Built-ins cannot dispatch further. The main agent's `[secondary_model]` pool governs which models subagents run on — “Subagents inherit the model the main agent is running by default,” and “Subagents only run on models listed in the pool.” Per-spawn `model` selection is available once the pool holds two or more entries.

## Custom agents and delegation

Agent files (Markdown + YAML frontmatter: `name`, `description` required; `whenToUse`, `override`, `tools`, `disallowedTools`, `subagents`) live in project `.kimi-code/agents/` and `.agents/agents/`, user `$KIMI_CODE_HOME/agents/` and `~/.agents/agents/`, plugins, or an explicit `--agent-file`. Precedence is “Explicit (--agent-file) > Project > Extra > User > Plugin > Built-in,” and “A project-scoped file can take over a built-in agent entirely” with `override: true`. Custom agents inherit the main agent's delegation allowlist: a custom subagent can only dispatch subagents the parent itself is allowed to dispatch. The `[subagent].timeout_ms` default is 7200000 (2 h).

The Agent tool spawns subagents with `subagent_type coder` by default, supports `run_in_background`, resume of prior subagents, a model-pool parameter, and a 2 h timeout; stopping a run returns incomplete text. AgentSwarm fans out over an item list with `{{item}}` templating — “The tool supports up to 128 total subagents, waits for all subagents to finish, and returns an aggregated report” — with rate-limited ramp-up (“5 subagents start immediately, then 1 more every 700 ms”) and a single call per assistant response.

## Tower (experimental)

Tower is the multi-agent coordinator behind `[experimental] tower = true` or `KIMI_CODE_EXPERIMENTAL_TOWER=1` set before launch. It plans missions (build missions with independent review, artifact missions with explicit acceptance, survey missions read-only), runs workers in isolated Git worktrees on branches, and merges only what passes review. Three modes are mutually exclusive with each other: Tower, Goal, and Plan.

Approval semantics are asymmetric: “Only the tower coordinator supports Ask When Needed. Workers and reviewers run automatically with Never Ask permissions and do not request approval for individual actions.” Scale and staffing: “Tower caps concurrently running workers and reviewers at a combined total of 256.” Delegation depth is shallow by design — “Tower workers do not have the Agent tool and cannot spawn subagents” — and the docs are explicit that isolation is process-level: “Worktrees isolate writes, but they are not a security sandbox. Workers can read repository content and run tools with your permissions,” and “Mission scopes and merge checks are process constraints, not security boundaries.”

Operational rules: “Each workspace can have only one tower coordinator”; after a restart “A project cannot have multiple tower coordinators at the same time,” and the new coordinator must re-spawn workers. Uncommitted work can be swept up — “Tower may snapshot uncommitted and untracked files into worker branches” — so “Check for secrets and large files first.” The coordinator owns Git: “Let Tower handle merges and rebases; do not run additional Git integration commands yourself. Do not edit files under .tower/comms/ directly: status, notifications, and merge checks depend on these records.” In noninteractive runs Tower “leaves the artifact waiting rather than approving it for you.” `/tower status|on|off|<base>|teardown` drives the lifecycle; teardown skips worktrees with uncommitted changes or running agents.

## Official sources

- [Agents](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/agents.html) · [Tools reference (Agent, AgentSwarm)](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/tools.html) · [Tower](https://www.kimi.com/code/docs/en/kimi-code-cli/tower.html) · [Model Configuration (secondary_model)](https://www.kimi.com/code/docs/en/kimi-code-cli/configuration/providers.html)
