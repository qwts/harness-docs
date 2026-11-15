# Automation: Agent Manager, cloud, reviews, and integrations

**Verified:** 2026-11-15

## Agent Manager (VS Code panel)

Agent Manager is the multi-session orchestration panel (open with Cmd+Shift+M). Managed worktrees “are created under `.kilo/worktrees/` in your project. Kilo also stores Agent Manager UI state in `.kilo/agent-manager.json`.” Each worktree branches from the project’s configured default base branch (“If no default is configured, Kilo falls back to automatic detection of the repository's remote default branch”).

- **Setup scripts** — `.kilo/setup-script`/`.sh` (macOS/Linux) and `.ps1`/`.cmd`/`.bat` (Windows), “checked in order”, run automatically on worktree creation via `sh`/PowerShell/`cmd.exe` (“so executable permissions are not required”). “The script keeps the existing five-minute timeout; when it expires, the setup process tree is terminated.” `WORKTREE_PATH` and `REPO_PATH` are injected into the script environment.
- **Env auto-copy** — “Before the setup script runs, Kilo automatically copies root-level `.env` files from the main repo into the new worktree”: root-level files named exactly `.env` or matching `.env.*`. Not copied: “Nested env files, such as `apps/web/.env.local`” and “Non-dotenv files, such as `.envrc`, `.environment`, or `.env-cmdrc`.”
- **Multi-Version Mode** — runs “up to 4” parallel implementations of one prompt in sibling worktrees; multi-version groups move between sections together.
- **agent_manager tool** — lets the agent drive the panel itself (worktree or local session modes, up to 20 tasks per request, `action: "list"` for IDs). “The tool uses the `agent_manager` permission. Approval prompts are scoped to the requested capability, so approving `worktree` does not automatically approve `local`, an overview, or a targeted prompt.” An agent answer “must use the question's exact option labels; approving a plan can start implementation.”
- **Kilo Swarm board** — agents coordinate via `board_post`/`board_read`; “Swarm does not start, stop, or assign agents.” Every stored post carries the receipt “Stored only. This does not confirm delivery, reading, or action, and does not wake recipients,” and results warn when the recipient’s invocation has ended or cannot be confirmed.
- **Projects** — Agent Manager resolves projects by “canonical Git repository root” (re-adding the same repo opens the existing project); “The VS Code window must be trusted before you can add, create, or clone a repository.” Sessions, worktrees, sections, and settings are per-project. A non-Git workspace folder stays the default project but “worktree actions on the default project fail.”
- **PR workflow** — PR association uses “the branch tracking ref, branch name, or current commit SHA” via the `gh` CLI (3 detection strategies); a Push Pull Request Fixes setting pushes agent fixes to PRs. The diff panel scopes to Branch/Staged/Unstaged/Session with Apply to local; sections organize worktrees; browser previews serve localhost/127.0.0.1 over HTTP only (system Chrome or bundled Playwright Chromium). Worktree creation isolates the agent: “The agent works in isolation, so your main branch is unaffected.” The workflows guide caps practical concurrency — “Do not run more than four or five agents at once” — and warns against “`git stash` inside a worktree. Stashes cross between worktrees.”

## Cloud Agents

Cloud Agents run Kilo in a remote environment that “can read and modify your GitHub, GitLab, or Bitbucket repositories” — “Bitbucket repositories are available for organizations only.” Sandbox pricing: Docker/Small at $0.60/hour and Standard at $1.20/hour, billed per second, with a $5 minimum balance. Vercel sandboxes are GitHub-only. `/remote` toggles Cloud Agent access from the CLI TUI; `kilo run --cloud-fork` and `/fork` create cloud sessions; Mobile (iOS/Android) drives the same agents.

## AI Code Reviews (GitHub/GitLab)

The Review Agent runs via the KiloConnect GitHub App (permissions: Pull requests Read & Write, Repository contents Read, Issues Read & Write, Metadata Read) or GitLab OAuth/PAT — GitLab requires “Maintainer role (or higher)” because Kilo creates a bot Project Access Token per project. Triggers: PR/MR opened, new commits pushed, reopened, draft marked ready; skipped for draft-first opens, closed, merged, closed PRs. Reviews announce with a 👀 reaction and post a summary comment plus inline comments tagged critical/warning/info; pushing new commits cancels the previous review and updates the summary in place. Default model Claude Sonnet 4.5; review styles Strict/Balanced/Lenient; focus areas configurable.

**REVIEW.md** — “Create REVIEW.md at the repository root. Commit it to the base branch used by pull requests or merge requests.” “Kilo reads REVIEW.md from the PR/MR base branch, not the feature branch. That prevents an unreviewed change from rewriting the review policy used to evaluate itself. If the file is disabled, missing, empty, or unreadable, Kilo falls back to built-in guidance. If it is longer than 10,000 characters, Kilo truncates it and notes that in the review summary footer.”

**Sub-agent tiers** — “By default, Code Reviews uses sub-agents only when they materially improve coverage. After reading the diff, the reviewer estimates changed file count and changed lines, then chooses the largest tier triggered by either signal”: ≤2 files and <100 lines → 0 sub-agents; 3–5 files or 100–300 lines → up to 1; 6+ files or >300 lines → “the full 6 sub-agents, sharded by independent areas.”

**Concurrency** — per owner: personal accounts with “credit balance of $5 or less” get 1, above $5 get up to 3 (“A funded personal account running a free model still gets 3 concurrent reviews”), organizations up to 20.

## GitHub bot and messaging platforms

Mentioning “@kilocode-bot on an issue, pull request, or review comment” dispatches a Cloud Agent that branches and opens a PR. It requires both the KiloConnect GitHub App and the Kilo Code Bot GitHub App — the docs’ own troubleshooting notes to check you’re “mentioning @kilocode-bot (not `@kilo`)”. Kilo Connect also bridges chat platforms: “@Kilo in any channel or DM” on Slack and “@kilo on any issue” in Linear. DoltHub OAuth integration covers “Dolt-versioned data” workflows through Kilo Connect.

## Official sources

- [Agent Manager](https://kilo.ai/docs/automate/agent-manager) · [Agent Manager projects](https://kilo.ai/docs/automate/agent-manager-projects) · [Workflows guide](https://kilo.ai/docs/automate/agent-manager-workflows) · [Cloud Agent](https://kilo.ai/docs/code-with-ai/platforms/cloud-agent) · [Code Reviews overview](https://kilo.ai/docs/automate/code-reviews/overview) · [GitHub](https://kilo.ai/docs/code-with-ai/platforms/github) · [GitLab](https://kilo.ai/docs/automate/code-reviews/gitlab) · [Kilo Connect](https://kilo.ai/docs/code-with-ai/platforms/kilo-connect) · [Integrations](https://kilo.ai/docs/automate/integrations)
