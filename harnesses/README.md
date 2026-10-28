# Active harness coverage

**Registry verified:** 2026-10-29  
**Source:** [`qwts/qwts-agent-org/governance/agents.json`](https://github.com/qwts/qwts-agent-org/blob/main/governance/agents.json)  
**Registry file SHA at verification:** `ae7d425b8fc5a1580ff28af50c038c69a51abdd6`  
**Coverage:** 29 active unique harnesses · 10 documented · 19 pending

## Method

- Start with identities whose registry status is active.
- Deduplicate identities by their `harness` value; multiple organization-specific identities using one harness produce one coverage row.
- Exclude retired entries, including retired model-specific identities, from active coverage.
- Treat a harness as documented only when it has a harness index in this repository.
- Registry notes describe the source organization's operations, not necessarily the vendor's product behavior. Product claims still require primary-source verification.

## Coverage table

| Registry harness | Product or project | Status | Documentation |
|---|---|---|---|
| `aider` | Aider | Pending | — |
| `amp` | Amp | Pending | — |
| `antigravity` | Google Antigravity | **Documented** | [Index](antigravity-agent-harness/README.md) |
| `claude-code` | Claude Code | **Documented** | [Index](claude-agent-harness/README.md) |
| `cline` | Cline | **Documented** | [Index](cline-agent-harness/README.md) |
| `codex` | OpenAI Codex | **Documented** | [Index](codex-agent-harness/README.md) |
| `command-code` | Command Code | Pending | — |
| `copilot` | GitHub Copilot | **Documented** | [Index](copilot-agent-harness/README.md) |
| `cursor` | Cursor | **Documented** | [Index](cursor-agent-harness/README.md) |
| `deepseek` | DeepSeek | Pending | — |
| `devin` | Devin | **Documented** | [Index](devin-agent-harness/README.md) |
| `factory-droid` | Factory Droid | Pending | — |
| `goose` | Goose | Pending | — |
| `grok` | Grok | Pending | — |
| `hermes` | Hermes | Pending | — |
| `junie` | Junie | Pending | — |
| `kilocode` | Kilo Code | Pending | — |
| `kimicode` | Kimi Code | Pending | — |
| `kiro` | Kiro | **Documented** | [Index](kiro-agent-harness/README.md) |
| `mcode` | MCode | Pending | — |
| `mirror` | Mirror | Pending | — |
| `muse` | Meta Muse Code | **Documented** | [Index](muse-agent-harness/README.md) |
| `opencode` | OpenCode | **Documented** | [Index](opencode-agent-harness/README.md) |
| `pi` | Pi | Pending | — |
| `qwen-code` | Qwen Code | Pending | — |
| `roocode` | Roo Code | Pending | — |
| `vibe` | Vibe | Pending | — |
| `warp` | Warp | Pending | — |
| `zcode` | ZCode | Pending | — |

## Maintenance rule

When the registry changes, recalculate active unique harnesses before adding documentation. Do not infer vendor configuration, file paths, or runtime behavior from registry identity notes; verify those claims against current official product documentation.
