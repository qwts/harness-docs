# Junie harness

**Verified:** 2026-11-15 against the official Junie documentation at junie.jetbrains.com/docs (JetBrains)
**Status:** Reference documentation, not a runtime conformance audit
**Primary scope:** `junie` CLI (TUI, headless, ACP), `~/.junie/` config and trust model, guidelines and memory, model providers (Junie/BYOK/custom/proxy), approvals and sandbox, MCP/skills/subagents/hooks/extensions, JetBrains IDE integration and plugin, GitHub/GitLab CI/CD
**Provenance:** Official JetBrains sources unless explicitly marked otherwise

Junie is JetBrains’ coding agent — "the agentic coding tool by JetBrains" — shipped as a standalone CLI (Linux, macOS, Windows), inside JetBrains IDEs, and on GitHub/GitLab via the Junie GitHub Action and GitLab CI/CD. The CLI installs with `curl -fsSL https://junie.jetbrains.com/install.sh | bash`, keeps its state under `~/.junie/` (`$JUNIE_HOME`), authenticates with a JetBrains account, a `JUNIE_API_KEY` token, BYOK provider keys, custom model profiles, or proxy endpoints, and layers four provider types (Junie, BYOK, Custom, Proxy) over model aliases like `sonnet` and `gpt`. Sensitive actions sit behind a three-level brave mode plus an editable Action Allowlist, with an opt-in OS-level sandbox mode wrapping agent terminal commands. A Claude-Code-shaped hooks system (EAP), MCP servers, agent skills, custom subagents, and a Claude-plugin-compatible extensions marketplace extend the agent. JetBrains positions Junie CLI in an Early Access Program, and several features (hooks, sandbox mode, subagents settings, custom proxies) are EAP-flagged on their own. The `junie` registry identity normalizes to this single harness; JetBrains AI Assistant is a related but separate product surface.

## Verdict legend

✅ TRUE · ⚠️ PARTIAL · ❌ FALSE · 💬 OPINION · ❓ UNVERIFIED

## Documentation map

| File | Coverage |
|---|---|
| [Configuration, guidelines, and trust](configuration-guidelines-and-trust.md) | `~/.junie/` layout, config.json channels and precedence, settings.json as a separate channel, project trust markers and keychain storage, environment variables, AGENTS.md discovery order, global guidelines, import from other agents, cache settings, themes |
| [Models, providers, and routing](models-providers-and-routing.md) | Four provider types, Default model and aliases, effort levels, provider auto-detection, primary vs faster model split, BYOK billing priority, custom LLM profiles, custom proxies (Ingrazzio/Bedrock/OpenAI/Google/Anthropic), Junie Local, Junie Lite, Eco mode |
| [Approvals, sandbox, and sessions](approvals-sandbox-and-sessions.md) | Brave mode levels, Action Allowlist rules and matching, sandbox mode (srt policy, network filter, git and SSH handling), parallel sessions and worktrees, plan mode, debug mode, local code review, teleport |
| [Tools, MCP, skills, and hooks](tools-mcp-skills-and-hooks.md) | MCP config and installation assistant, per-server timeout, agent skills and `$` references, custom subagents and tool groups, hooks (8 events, EAP) with Claude-Code-compatible shapes, extensions and marketplaces |
| [Surfaces: CLI, IDE, and CI/CD](surfaces-cli-ide-and-cicd.md) | Install and authentication, interactive TUI, headless mode, JetBrains IDE integration and the Junie plugin, GitHub Action (workflows, parameters, MCP servers), GitLab CI/CD, Jira/YouTrack integrations, ACP, remote mode, demo agent, EAP and cache management |
| [Corrections](corrections.md) | Claim-by-claim misinformation audit |

## Top corrections

- ❌ **“Junie stacks `.junie/AGENTS.md` and `AGENTS.md` together.”** When `.junie/AGENTS.md` exists it “is used exclusively; no other guidelines files are combined with it”; the AGENTS.md + playbook + rules combination is the fallback path.
- ❌ **“Hooks can ship in the project’s `.junie/config.json`.”** Project-local hooks from the default project config “are ignored by default for safety”; only user config or explicit `--config-location` files contribute hooks.
- ❌ **“Sandbox mode isolates MCP servers and hooks too.”** “MCP servers, hooks, and operations performed directly by Junie CLI outside those tools are not isolated by this sandbox… It is not hardened isolation like a container or VM.”
- ❌ **“The sandbox blocks network access by default.”** “Network access is unrestricted by default”; filtering is opt-in via `~/.junie/sandbox.json` (`"unrestricted": false` + `allowedDomains`), and only the global policy file is used — a project-local one is ignored.
- ❌ **“CI must pre-seed a trust marker before Junie runs.”** “Non-interactive JSON, ACP, and Gateway launches are always trusted: they cannot ask you for a decision, so they load project configuration without a prompt, a flag, or an environment variable.”
- ❌ **“Stop hooks can block the agent indefinitely.”** “The agent stops dispatching the Stop hook after 8 consecutive blocks within the same task” — overridable via `JUNIE_STOP_HOOK_BLOCK_CAP` (0 disables the cap).
- ❌ **“`pull-requests: write` is enough for the GitHub Action to open PRs.”** The repo must also enable “Allow GitHub Actions to create and approve pull requests” in Settings → Actions → General — “With this setting disabled, the Junie GitHub action will not be able to create PRs.”
- ❌ **“Remote mode works with BYOK alone.”** It “requires a Junie subscription” — a JetBrains Account or `JUNIE_API_KEY`; “Bring Your Own Key (BYOK) on its own is not enough,” and it is unavailable under JetBrains AI Enterprise.
- ❌ **“With a JetBrains subscription present, usage is billed to JetBrains first.”** “If a model is available through both your BYOK key and your JetBrains subscription, the BYOK key takes priority, and the requests are billed to your provider.”

## Quick reference

| Concern | Verified location or behavior |
|---|---|
| Home | `~/.junie/` (`$JUNIE_HOME` overrides); trust markers in `~/.junie/trust`; sessions under `~/.junie/sessions/` |
| Config channels | `config.json` (user + project, merged) and `~/.junie/settings.json` — “There is no single global precedence across both channels” |
| config.json precedence | CLI flags > project config (trusted project) > user config; relative paths resolve against the file’s folder |
| Extra configs | `--config-location` (repeatable); `--config-default-locations false`; `JUNIE_CONFIG_LOCATION` / `JUNIE_CONFIG_DEFAULT_LOCATIONS` |
| Trust | Interactive sessions prompt (untrusted / trust project / trust parent); keychain-backed auth key (Keychain / Credential Manager / Secret Service, owner-only file fallback); canonicalized paths, symlink-resolved |
| Untrusted projects | File operations allowed; project config, MCP, hooks, extensions, models, skills, agents, guidelines, memory not loaded; writable temp Junie dir outside the repo, cleared at exit |
| Guidelines | `.junie/AGENTS.md` exclusive → else root `AGENTS.md` + `.junie/playbook.md` + `.junie/rules/*.md` → legacy `.junie/guidelines.md`; global `~/.junie/AGENTS.md` combined (project wins conflicts, identical content deduped) |
| Custom guidelines | `--guidelines-filename` / `JUNIE_GUIDELINES_FILENAME` (file placed in `.junie/`); IDE Project Settings Guidelines path |
| Models | Four provider types: Junie, BYOK, Custom, Proxy; `Default` is a dynamic best-price-quality pick; aliases (`sonnet`, `opus`, `fable`, `gpt*`, `gemini-*`, `grok`) may change |
| Effort | `minimal`–`max` per model; `--effort`/`JUNIE_EFFORT` requires `--model`/`JUNIE_MODEL`/`--agent`; `/model`, `/effort` |
| Internal model | Helper tasks (summarize, classify, memory extraction) use a separate cheaper model “from the same provider… even when using BYOK” |
| BYOK | OpenAI, Anthropic, Google, xAI, OpenRouter (API keys), GitHub Copilot (local Copilot CLI ≥1.0.65 OAuth); BYOK beats subscription billing for the same model |
| Custom models | JSON profiles in `$JUNIE_HOME/models/` and `.junie/models/`; `apiType`: OpenAICompletion / OpenAIResponses / Google / Anthropic; `${VAR}` env refs in `apiKey`/`extraHeaders`; `custom:<profile-id>` selector |
| Proxies | config.json `proxies` (Ingrazzio default kind; Bedrock, OpenAI, Google, Anthropic supported; JetBrainsAI/OpenRouter reserved); legacy `INGRAZZIO_URL` env |
| Junie Local | Local inference engine — macOS Apple Silicon (M4+, 48 GB, Splash/MLX) or Windows NVIDIA RTX (24 GB VRAM, llama.cpp CUDA); 4-bit Qwen3.6-27B / Qwen3.8-3.6-27B-blend; OpenAI-compatible server on `localhost:19239` |
| Junie Lite | Free standing tier (currently Qwen 3.6 27B, may change without notice); “virtually unlimited”; no downgrade path; not in JetBrains AI Assistant |
| Eco mode | Plans with the selected model, implements with a Junie-chosen cheaper model from the same provider route; interactive/ACP only; not headless, not remote-bridge |
| Brave mode | Off / Auto / On via `/brave` or Ctrl+B; Auto classifies commands with a safety check; `--brave` turns it on (interactive only) |
| Allowlist | `~/.junie/allowlist.json`; CLI types: fileEditing, executables, mcpTools, readOutsideProject, readSecretFile; prefix + glob pattern rules, first match wins; chained/nested/multi-line commands re-checked |
| Sandbox | `junie --sandbox` / `/sandbox` (EAP; macOS/Linux only); srt wraps terminal commands and restricts native edit tools; project + git + temp + tool-cache writes only; `~/.junie/sandbox.json` policy; `~/.ssh` deny-read except config/known_hosts |
| Sandbox limits | MCP servers, hooks, and direct Junie CLI operations not isolated; srt’s own deny-writes: `.git/hooks`, `.git/config`, shell rc files, `.mcp.json`, `.vscode`/`.idea` |
| MCP | `mcpServers` JSON in `.junie/mcp/mcp.json` (project) and `~/.junie/mcp/mcp.json` (user); `/mcp` manager + Installation Assistant; OAuth authorize flow; per-server `timeout` ms field (1,000–3,600,000; default 60 s) |
| MCP scope (IDE) | “Junie supports a maximum of 100 tools from all configured MCP servers” (IDE plugin docs) |
| Skills | `.junie/skills/` (project) and `~/.junie/skills/` (user) + `.agents/skills/` convention + extensions + built-ins; SKILL.md with name/description frontmatter; `/<skill-name>` commands and `$<skill-name>` references; project beats user on name conflicts |
| Subagents | Markdown + YAML frontmatter in `.junie/agents/`, `.agents/`, user equivalents; auto-delegation by name/description match; `--agent <name>` standalone; policy SameModelOnly / Auto (default) in settings.json (EAP) |
| Hooks | EAP; 8 events (SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop, StopFailure, PermissionRequest, SessionEnd) in `~/.junie/config.json` `hooks`; only `type: "command"`; exit 2 blocks; `blockOnError`/`strictJsonOutput`/`async` Junie extensions; Stop block cap 8 |
| Hook compat | Field names follow Claude Code wire protocols (StopFailure `error`/`error_details`; PreToolUse `decision`/`tool_input`); extensions may ship Claude-plugin `hooks/hooks.json` with `${CLAUDE_PLUGIN_ROOT}` |
| Extensions | `/extensions` (aliases `/plugin`, `/plugins`); marketplaces as git repo / local dir / URL; native `.junie-extension/marketplace.json` + Claude `.claude-plugin/marketplace.json`; skills+MCP+subagents+commands+guidelines+hooks bundles |
| Slash commands | ~40 built-ins (`/new`, `/plan`, `/review`, `/mcp`, `/worktree`, `/teleport`, `/goal`, `/agent-mode`, `/compress`, `/stats`, …); custom commands as Markdown in `.junie/commands/` + `~/.junie/commands/`; `$arg` parameters; `allowPromptArgument: true` for free text |
| Modes | Classic vs Chat (`/agent-mode`, locked after first prompt); Plan mode (Shift+Tab / `/plan` / `--plan`, applies to next prompt only); Debug mode (IDE-backed, `/debug`) |
| Sessions | `/new` keeps current live in background; `/history` Task history; `Ctrl+O` transcript (transcript.md next to events.jsonl); `/branch`, `--fork-session`, `--session-id` + `junie://sessions/<id>` links, `/teleport` + `/continue` |
| Worktrees | `/worktree` menu; created as siblings `<project>-junie-wt-NN`; uncommitted changes transferred via git stash or clean start; git repos only |
| Review | `/review [target]` local review via read-only review subagent; `--review` headless; same backend as GitHub automated reviews; `[CRITICAL]` findings |
| Headless | `junie --auth "$JUNIE_API_KEY" "task"`; `--task`/positional; `--output-format text\|json\|json-stream`; `--input-format`; `--act-as-interactive`; JSON `codeReviewTask` input with `reviewTarget`/`diffCommand`/`allowApprove` |
| GitHub Action | `JetBrains/junie-github-action@v1`; @junie-agent mentions, label/assignee triggers; outputs branch_name/commit_sha/pr_url/junie_summary; `mcp_github_checks_server` + `mcp_github_inline_comment_server` (auto for PRs); BYOK keys alternative, “Provide either junie_api_key or one of the BYOK keys — not both” |
| GitLab | Separate container project per instance; `#junie` mention or bot tag; MR auto-review via `JUNIE_CUSTOM_PROMPT`; webhook + access token wiring |
| Jira/YouTrack | Jira automation rules → `workflow_dispatch` (@junie comments, label triggers); YouTrack app adds Run Junie widget + trigger comments |
| ACP | `junie --acp true` (JSON-RPC over stdio local); `/mcp` read-only, `/sandbox on\|off`, limited command set; V1 vs V2 command advertising |
| Remote | `/remote` tunnels the live session to junie.jetbrains.com/remote/<id>; one web client at a time; requires JetBrains account or API key |
| Demo | `/demo` builds the app in a disposable Docker VM (`registry.jetbrains.team/p/junie-cli/containers/demo-base:2`); needs Docker + Computer-Use model; scopes: branch/session/uncommitted/last commit/smoke test |
| IDE integration | `/ide` status; 10 supported JetBrains IDEs; passive CLI-side discovery; symbol-aware search, inspections, test runs, `@` completions; IDE 2026.1+ for “Unsupported” fix |
| IDE plugin | Recommended surface is the single AI Chat (AI Assistant plugin, auto-download); manual plugin only for a separate tool window; Code/Ask modes; `.aiignore`; Project Settings; BYOK/model selection not via plugin |
| IDE allowlist | Settings \| Tools \| Junie \| Action Allowlist; rule types Terminal/RunTest/Build/Preview/MCP/Read outside project/Write outside project/Edit build scripts/Edit configuration or hidden files/Read files with secrets; ls/cd/pwd free by default; MCP rule allows all tools |
| Cache | `junie cache [clean]`; versions (keep 3), update archives (7 d), logs (30 d); sessions never auto-deleted without `--sessions` / `cacheAutoPruneSessions` (90 d, finished only); no pruning headless |
| Install | `curl -fsSL https://junie.jetbrains.com/install.sh \| bash`; PowerShell install.ps1; EAP via install-eap.sh; managed launcher (`junie` shim) required for `/restart`; channels release/eap/nightly/experimental |
| Env vars | JUNIE_API_KEY, JUNIE_ANTHROPIC/OPENAI/GOOGLE/GROK/META/OPENROUTER_API_KEY, JUNIE_LITELLM_URL/API_KEY, JUNIE_MODEL, JUNIE_LLM_PROVIDER, JUNIE_EFFORT, JUNIE_TASK/PROMPT/SYSTEM_PROMPT/PROJECT, JUNIE_HOME, JUNIE_CONFIG_*, JUNIE_*_LOCATIONS, JUNIE_AGENT, JUNIE_STOP_HOOK_BLOCK_CAP |

## Official sources

- [Junie docs home](https://junie.jetbrains.com/docs/) · [Quickstart (Junie CLI)](https://junie.jetbrains.com/docs/junie-cli.html) · [CLI reference](https://junie.jetbrains.com/docs/parameters.html) · [Environment variables](https://junie.jetbrains.com/docs/environment-variables.html) · [Slash commands reference](https://junie.jetbrains.com/docs/slash-commands.html)
- [config.json](https://junie.jetbrains.com/docs/junie-cli-configuration.html) · [Guidelines and memory](https://junie.jetbrains.com/docs/guidelines-and-memory.html) · [Model selection](https://junie.jetbrains.com/docs/junie-cli-model-selection.html) · [BYOK](https://junie.jetbrains.com/docs/byok.html) · [Custom LLMs](https://junie.jetbrains.com/docs/custom-llm-models.html) · [Custom proxies](https://junie.jetbrains.com/docs/custom-proxies.html)
- [Add commands to Action Allowlist (CLI)](https://junie.jetbrains.com/docs/action-allowlist-junie-cli.html) · [Plan mode](https://junie.jetbrains.com/docs/junie-cli-plan-mode.html) · [Debug mode](https://junie.jetbrains.com/docs/junie-cli-debug-mode.html) · [Parallel sessions and worktrees](https://junie.jetbrains.com/docs/junie-cli-worktrees.html) · [Remote mode](https://junie.jetbrains.com/docs/junie-cli-remote-mode.html) · [Headless mode](https://junie.jetbrains.com/docs/junie-headless.html)
- [MCP configuration](https://junie.jetbrains.com/docs/junie-cli-mcp-configuration.html) · [Agent skills](https://junie.jetbrains.com/docs/agent-skills.html) · [Custom subagents](https://junie.jetbrains.com/docs/junie-cli-subagents.html) · [Hooks](https://junie.jetbrains.com/docs/junie-cli-hooks.html) · [Extensions](https://junie.jetbrains.com/docs/junie-cli-extensions.html) · [Custom slash commands](https://junie.jetbrains.com/docs/custom-slash-commands.html)
- [Junie GitHub Action](https://junie.jetbrains.com/docs/junie-on-github.html) · [Basic workflow](https://junie.jetbrains.com/docs/basic-workflow.html) · [Automated code reviews](https://junie.jetbrains.com/docs/automated-code-reviews.html) · [Code review agent](https://junie.jetbrains.com/docs/junie-review-agent.html) · [Junie GitLab CI/CD](https://junie.jetbrains.com/docs/junie-gitlab-ci-cd.html)
- [Integration with JetBrains IDEs](https://junie.jetbrains.com/docs/junie-cli-jetbrains-ide-integration.html) · [Junie IDE plugin](https://junie.jetbrains.com/docs/junie-ide-plugin.html) · [Junie for ACP clients](https://junie.jetbrains.com/docs/junie-cli-acp.html) · [Junie Local](https://junie.jetbrains.com/docs/junie-local.html) · [Junie Lite](https://junie.jetbrains.com/docs/junie-lite.html) · [Junie Eco mode](https://junie.jetbrains.com/docs/junie-eco.html) · [Early Access Program](https://junie.jetbrains.com/docs/junie-cli-eap.html) · [Cache management](https://junie.jetbrains.com/docs/junie-cli-cache.html)
