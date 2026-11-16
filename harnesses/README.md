# Active harness coverage

**Registry verified:** 2026-11-15  
**Source:** [`qwts/qwts-agent-org/governance/agents.json`](https://github.com/qwts/qwts-agent-org/blob/main/governance/agents.json)  
**Registry file SHA at verification:** `5c6de133c489f103e1dc9aa1cd280ca6de6de537`  
**Coverage:** 29 active unique harnesses · 14 documented · 15 pending

## Method

- Start with identities whose registry status is active.
- Deduplicate identities by their `harness` value; multiple organization-specific identities using one harness produce one coverage row.
- Exclude retired entries, including retired model-specific identities, from active coverage.
- Treat a harness as documented only when it has a harness index in this repository.
- Registry notes describe the source organization's operations, not necessarily the vendor's product behavior. Product claims still require primary-source verification.

## Coverage table

| Registry harness | Product or project | Status | Documentation |
|---|---|---|---|
| `aider` | Aider | **Documented** | [Index](aider-agent-harness/README.md) |
| `amp` | Amp | **Documented** | [Index](amp-agent-harness/README.md) |
| `antigravity` | Google Antigravity | **Documented** | [Index](antigravity-agent-harness/README.md) |
| `claude-code` | Claude Code | **Documented** | [Index](claude-agent-harness/README.md) |
| `cline` | Cline | **Documented** | [Index](cline-agent-harness/README.md) |
| `codex` | OpenAI Codex | **Documented** | [Index](codex-agent-harness/README.md) |
| `command-code` | Command Code | **Documented** | [Index](command-code-agent-harness/README.md) |
| `copilot` | GitHub Copilot | **Documented** | [Index](copilot-agent-harness/README.md) |
| `cursor` | Cursor | **Documented** | [Index](cursor-agent-harness/README.md) |
| `deepseek` | DeepSeek | **Documented** | [Index](deepseek-agent-harness/README.md) |
| `devin` | Devin | **Documented** | [Index](devin-agent-harness/README.md) |
| `factory-droid` | Factory Droid | **Documented** | [Index](factory-droid-agent-harness/README.md) |
| `goose` | Goose | **Documented** | [Index](goose-agent-harness/README.md) |
| `grok` | Grok | **Documented** | [Index](grok-agent-harness/README.md) |
| `hermes` | Hermes | **Documented** | [Index](hermes-agent-harness/README.md) |
| `junie` | Junie | **Documented** | [Index](junie-agent-harness/README.md) |
| `kilocode` | Kilo Code | **Documented** | [Index](kilocode-agent-harness/README.md) |
| `kimicode` | Kimi Code | **Documented** | [Index](kimicode-agent-harness/README.md) |
| `kiro` | Kiro | **Documented** | [Index](kiro-agent-harness/README.md) |
| `mcode` | MCode | **Documented** | [Index](mcode-agent-harness/README.md) |
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
