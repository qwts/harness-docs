# Agents, permissions, and sandbox

**Verified:** 2026-11-15

## Built-in agents

Kilo ships these built-in agents:

| Agent | Behavior |
|---|---|
| `code` (default) | “A skilled software engineer with expertise in programming languages, design patterns, and best practices” — “Full access to all tools: `read`, `edit`, `glob`, `grep`, `bash`, `task`, `webfetch`, plus tools from MCP servers”; “No tool restrictions” |
| `ask` | Read-only Q&A — read-only tools, “read-only bash commands (`cat`, `grep`, `git log`, `git diff`, `jq`, etc.)”; “All write operations are blocked”; MCP tools “with user approval” per call |
| `plan` | Read-only tools “plus restricted file editing (plan files in `.kilo/plans/` only)”; “Similar to the legacy extension's "Architect" mode” |
| `debug` | “Full access to all tools”; systematic troubleshooting focus |
| `orchestrator` | “Deprecated and will be removed in a future release. Agents with full tool access (Code, Plan, Debug) now support subagents natively” |
| `explore`, `general` | Additional built-in names (overrideable) — read-only codebase exploration and general-purpose delegation |

Review workflows have no dedicated built-in: “The VSCode extension and CLI do not include a built-in Review agent. Code review workflows can be handled by the code agent or via custom agent configurations.”

## Custom agents

Custom agents are Markdown files with YAML frontmatter. Active locations: `.kilo/agents/` (project), `~/.config/kilo/agent/` (global); legacy `.kilocode/agents/`, `custom_modes.yaml`, and `.kilocodemodes` are auto-migrated, with the old `build`/`architect` modes renamed to `code`/`plan`. Frontmatter keys: `description`, `model` (`provider/model`), `mode` (`primary`, `subagent`, `all`), `permission` (per-agent tool overrides), `color` (hex or theme keyword), `steps`, `temperature`/`top_p`, `variant`, `hidden`, `disable`.

Agent configurations merge “from lowest to highest priority”: built-in defaults → global `~/.config/kilo/kilo.jsonc` → project `kilo.jsonc` → `.kilo/` / legacy `.kilocode/` directory configs and agent `.md` files → `KILO_CONFIG_CONTENT`. “When the same agent name appears at multiple levels, properties are merged (not replaced wholesale).” Any built-in (`code`, `plan`, `debug`, `ask`, `orchestrator`, `explore`, `general`) can be overridden by defining an agent with the same name. Organization-managed agents can shadow built-ins too: “An organization-managed agent can use the same name as a built-in agent. When it does, the organization-provided definition takes precedence for members of that organization,” and members cannot remove them locally.

## The permission engine

Permissions are `allow` / `ask` / `deny` per tool, configured in `kilo.jsonc` or per-agent frontmatter. “Permission rules are evaluated in config order. When more than one rule matches the requested permission and target pattern, the last matching rule wins.” Permitted tools span `external_directory`, `bash`, `read`, `edit`, `glob`, `grep`, `task`, `agent_manager`, `skill`, `lsp`, `todoread`/`todowrite`, `websearch`, `webfetch`, and `doom_loop`.

**Shipped defaults:** “Most tools default to `"*": "allow"` for a smooth out-of-the-box experience. Notable exceptions that prompt by default:”

- `.env` files — reading them prompts; `*.env.*` (e.g. `.env.local`, `.env.production`) also asks, while `*.env.example` is explicitly allowed. Broad `read: allow`, saved wildcard approvals, and allow-everything mode “do not bypass the built-in prompt for these files.”
- `external_directory` — accessing files outside the project prompts.
- `doom_loop` — “prompts when the agent enters a repeated failure cycle.”
- `edit` (JetBrains plugin only) — file edits prompt; “This is a plugin default, not an environment override.”

When no rule matches at all, “the default action is ask.” (An older docs page asserts “No tools are auto-approved out of the box” — contradicted by the current Auto-Approve reference; see corrections.) Runtime approvals happen in the Permission Dock: approve once, approve always (saves an allow rule to config), or deny.

**Shell rules:** “The bash permission is checked against parsed shell command patterns. If a shell block contains multiple parsed commands, each relevant command must be permitted. A single denied command rejects the request.” Commands are parsed with Tree-sitter, enabling path resolution, external-directory detection, and structured intent analysis. Built-in read-only agents additionally restrict write-like shell patterns (redirection, command substitution, pipes, chains).

**MCP tools** use the same system with keys `{server}_{tool}` (e.g. `github_create_pull_request`) and glob patterns like `github_*`.

## Sandbox

The OS-level sandbox is **disabled by default** and “does not restrict filesystem reads. An agent can still read any file that your user account can read, but it can write only to explicitly allowed locations.” “Sandboxing is not available on Windows. If an enabled macOS or Linux sandbox policy cannot be enforced, Kilo reports the reason and refuses to run the affected tool. It does not silently fall back to unrestricted execution.”

Network policy defaults to `"deny"`; `sandbox.allowed_hosts` whitelists DNS hosts with optional port (default 443) — “Wildcards, URLs, IP literals, private or reserved addresses, and implicit subdomains are rejected.” With allowed hosts set, “Kilo keeps direct sockets blocked and exposes an authenticated proxy as the only supported egress path.” `sandbox.allowed_hosts` and `sandbox.writable_paths` can only be set in global config; project config may tighten but never widen. “SSH Git remotes are not supported.”

Hard boundaries worth knowing: “`.git` directories are always read-only to sandboxed tools” — mutating git commands in a linked worktree prompt because “the .git file points into the parent repository's .git directory. That directory is outside the worktree, so the write is outside the sandbox write boundary.” “Read-only git commands, such as `git status` or `git log`, do not trigger the prompt.” While network restriction is active, “local and remote MCP tool calls and typed resource reads are unavailable, but MCP prompt retrieval remains outside the session network policy.” The docs are explicit that this is not a privacy wall: “The network sandbox is not a provider privacy control. Provider and model inference traffic remains available. If Kilo reads a secret and includes it in a prompt, tool result, or conversation context, that content may be sent to the configured model provider even while network restriction is on.”

## Cost controls

- **Doom-loop protection** — “When the agent enters a repeated failure cycle — attempting the same action multiple times without making progress — Kilo pauses and asks for permission before continuing. This is controlled by the `doom_loop` permission, which defaults to ask.”
- **Balance hard stop** — gateway paid-model requests return HTTP 402 with a `buyCreditsUrl` link once the balance hits zero.
- **Per-user daily limits** — “When a member reaches their daily cap, subsequent requests are blocked until midnight UTC, when the limit resets.”
- **Spend alerts** — a spending-limit alert “fires when spend in a rolling 24-hour, 7-day, or 30-day window crosses the limit you set”; a spike alert “fires when one hour's spend runs a set multiple (1x to 50x) above the scope's usual rate, taken from the previous 14 days of hourly usage. The rule stays quiet until the scope has a day of history.” Notifications arrive by email and mobile push.

## Official sources

- [Using agents](https://kilo.ai/docs/code-with-ai/agents/using-agents) · [Custom modes/agents](https://kilo.ai/docs/customize/custom-modes) · [Agent permissions](https://kilo.ai/docs/customize/agent-permissions) · [Auto-approving actions](https://kilo.ai/docs/getting-started/settings/auto-approving-actions) · [Sandboxing](https://kilo.ai/docs/getting-started/settings/sandboxing) · [Cost controls & usage safeguards](https://kilo.ai/docs/getting-started/cost-controls-and-usage-safeguards)
