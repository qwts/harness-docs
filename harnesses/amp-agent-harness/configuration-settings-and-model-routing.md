# Configuration, settings, and model routing

**Verified:** 2026-11-15

## Settings files and precedence

| File | Location |
|---|---|
| User settings | `~/.config/amp/settings.json` or `.jsonc` (macOS/Linux); `%USERPROFILE%\.config\amp\settings.json` (Windows) |
| Workspace settings | The nearest `.amp/settings.json` or `.jsonc`, searched upward from the current directory to the repository root |
| Custom user settings | Any path passed with `--settings-file` |
| Workspace policy document | One JSON document in Workspace Settings → Managed Settings (cloud-hosted; each save is a revision) |
| Managed settings (machine-level) | `/Library/Application Support/ampcode/managed-settings.json` (macOS), `/etc/ampcode/managed-settings.json` (Linux), `%ProgramData%\ampcode\managed-settings.json` (Windows) |

`amp config edit` opens the user settings (`--workspace` for workspace settings), and `"$schema": "https://ampcode.com/cli-settings.schema.json"` enables validation. When the same setting appears in multiple places, workspace settings override user settings — with one documented reversal: keymap entries in user settings override workspace entries. Every key is namespaced under the `amp.` prefix.

Two managed layers sit above those files. Administrators add one JSON policy document in Workspace Settings → Managed Settings — each save creates a revision recording the author and time. When the workspace policy document and local settings set the same scalar, the workspace value wins; lists are combined with workspace entries first, and objects merge key by key with workspace values winning conflicts. A machine-level `managed-settings.json` merges in the same way and takes precedence over the workspace policy document. The blocked keys — `amp.url`, `amp.proxy`, `amp.mcpServers`, `amp.workspaces`, `amp.mcpTrustedServers` — apply to the admin policy document only: repository `.amp/settings.json` can set `amp.mcpServers` (subject to the MCP approval flow), and `amp.mcpServers` at the policy layer is replaced by the server MCP registry and hosted MCP servers instead.

## Notable settings keys

| Key | Default | Purpose |
|---|---|---|
| `amp.showCosts` | `true` | Thread cost info in the CLI |
| `amp.mcpServers` | — | MCP servers exposed to the agent |
| `amp.mcpPermissions` | `[]` | Allow/reject rules for MCP servers; the first matching rule applies and unmatched servers are allowed |
| `amp.tools.disable` | `[]` | Disable tools by name, with glob support and `builtin:toolname` syntax |
| `amp.git.commit.ampThread.enabled` | `true` | Adds an `Amp-Thread-ID: <thread-url>` trailer to commits |
| `amp.git.commit.coauthor.enabled` | `true` | Adds `Co-authored-by: Amp <amp@ampcode.com>` |
| `amp.skills.path` | — | Colon-separated extra skill directories (semicolons on Windows) |
| `amp.skills.disableClaudeCodeSkills` | `false` | Skip `.claude/skills/` and other Claude-compatible locations |
| `amp.skills.disableGlobalAgentsSkills` | `false` | Skip `~/.config/agents/skills/` and `~/.agents/skills/` |
| `amp.notifications.enabled` | `true` | Sounds on task completion/blocking |
| `amp.updates.mode` | `"auto"` | `auto`, `warn`, or `disabled` |
| `amp.runner.autoUpdate.enabled` | `true` | Runners install CLI updates |
| `amp.thread.autoArchiveOnQuit` | `false` | Archive open threads on quit |

## Accounts, keys, and updates

`amp login` adds an account per Amp server; `amp account list|switch|logout` manages them, and running processes keep the account they started with. `AMP_API_KEY` “takes precedence over saved accounts until you unset it.” Execute-mode CI tokens must start with `sgamp_`; `amp login` session tokens are rejected in that role because they expire within an hour.

The CLI updates itself in the background — “You don’t have to do anything except regularly restart Amp” — with `amp update` for a manual check, `amp version` for the version, and `AMP_SKIP_UPDATE_CHECK=1` to disable checks entirely. Other environment variables include `AMP_DISABLE_AMP_THREAD_TRAILER=1`, `AMP_FORCE_BEL`, and the standard `HTTP_PROXY`/`HTTPS_PROXY`/`NODE_EXTRA_CA_CERTS` networking set.

## Model routing

“By default Amp serves every model and bills your Amp credits.” Users can add connections so specific models are served — and billed — elsewhere. Four connection kinds are documented: provider API keys (Anthropic, Google AI Studio, Meta AI, OpenAI, xAI; OpenAI/Anthropic/OpenRouter also support workload identity federation), browser sign-in subscriptions (ChatGPT, 𝕏 Premium+/SuperGrok), cloud platforms (Amazon Bedrock, Google Cloud Agent Platform/Vertex, Microsoft Foundry), and gateways (OpenRouter, Vercel/Cloudflare/ngrok AI Gateways, Ollama Cloud, OpenCode Go, or any OpenAI-/Anthropic-compatible endpoint via a Custom URL connection).

Routing order: personal connections top-to-bottom, then workspace connections, then Amp — the first active connection whose model mapping includes the model wins; everything uncovered is served by Amp. Priority `0` is tried first, and subscriptions allow only one active connection of each type per user. “Sign in with ChatGPT also signs you in to Amp,” so one ChatGPT account maps to one Amp account. GitHub Copilot is a per-person connection, “not everyone in their Amp workspace,” and the credential is kept out of orb environments.

Networking is a hard constraint: “Amp sends model requests from its own servers,” so providers that allowlist caller IPs must allow Amp’s published egress IP addresses. Painter image generation requires a Google AI Studio API key connection and is billed by Google, not a Gemini consumer subscription.

## Official sources

- [CLI settings](https://ampcode.com/docs/cli/settings)
- [CLI](https://ampcode.com/docs/cli)
- [Model routing](https://ampcode.com/docs/customize/model-routing)
- [Egress IPs](https://ampcode.com/docs/egress-ips)
- [Execute mode](https://ampcode.com/docs/cli/execute-mode)
