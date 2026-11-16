# Installation, connection, and model access

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; app v3.18.2 “· Latest”, changelog release 2026-11-13)

## What ZCode is

“ZCode is an Agentic Development Environment (ADE) built to bring GLM-5.3 into real coding workflows,” giving GLM-5.3 “a stable desktop experience for planning, coding, reviewing, and iterating across complex development tasks.” It “has been deeply tuned and specifically optimized around GLM-5.3, with a focus on the first-party ZCode Agent,” and “Model, tools, and execution workflow are more tightly integrated.” The docs title names it the official harness: “ZCode | GLM-5.3 官方 Harness.”

Surfaces: the desktop workspace, mobile **Remote** (“desktop, mobile Remote, and Feishu / WeChat Bot can all keep the same workspace task moving”), and the desktop-only built-in browser panel. Only the desktop app is documented — no CLI/headless surface (see [integrations, automation, and remote](integrations-automation-and-remote.md)).

## Install

“Download and install the ZCode desktop app.” Latest download at verification: **v3.18.2**. “Supports macOS (Apple Silicon / Intel), Windows (x64 / ARM64), and Linux (x64, ARM64, AppImage, DEB, RPM).” Install is drag-to-Applications (macOS), setup wizard (Windows), or AppImage/deb/rpm (`chmod +x ZCode-*.AppImage`) on Linux.

First launch: the onboarding page ends with “click Connect in the bottom-left corner to reach the sign-in page.” The **Data Migration Wizard** “currently only imports conversations from Claude Code and ZCode Agent in legacy ZCode — other tools are not supported” (can be deferred to Settings). Then pick a project directory as the workspace and send a test instruction.

Windows specifics: “Under Settings → General → Terminal you can pick which shell the Agent uses to run commands: Auto, CMD, or Git Bash (shown only when it's installed). Auto is the default and prefers Git Bash, falling back to cmd.exe. Changes apply to new sessions only.” Since v3.4.0, closing the window hides to tray by default — “which is also what lets scheduled and idle-time tasks keep running in the background” — and the disable toggle lives under Settings → General → Notifications.

Troubleshooting anchors: macOS quarantine via `xattr -dr com.apple.quarantine /Applications/ZCode.app`; Linux needs `libfuse2` for AppImage; browser login returns through the **`zcode://`** URL scheme, so a broken handler (`xdg-mime query default x-scheme-handler/zcode`) breaks sign-in on Linux.

## Network proxy

Three fields under Settings → General (save and restart): **HTTP Proxy** — “Empty doesn't mean “follow the system.”” When this field is blank “ZCode connects directly and does not read HTTP_PROXY or similar environment variables. That's the opposite of how many command-line tools behave”; once set, “model requests, MCP servers, command-line tools the Agent runs, and the app's own interface requests all go through this proxy.” **No proxy** — “Hostnames, domain suffixes like `.example.com`, and wildcards like `*.corp.com` all work, and you can include a port.” **Custom certificate** — “the local path to a PEM root certificate … this is not a way to skip certificate validation: ZCode only accepts connections whose chain actually contains that root certificate.”

## Connecting models

“ZCode supports multiple ways to access GLM models for coding workflows.” Three entry paths (first-launch welcome screen, “Manage Models” at the bottom of the model picker, or Connect on sign-in):

- **Connect Z.ai** (“Recommended” for global users) — “Sign in with your Z.ai account to use GLM models in ZCode. If your account has a GLM Coding Plan, ZCode uses the plan and quota from that account directly.”
- **Connect BigModel** (China) — subscribe via the BigModel open platform. New users get a trial: “no payment required, with 8 million tokens per day for the first 5 days,” split “GLM-5.3 3 million tokens / day” and “GLM-5.3-Flash 5 million tokens / day.” “The trial lasts 5 days only … it is not an ongoing daily allowance.” GLM Coding plans (Lite/Pro/Max) can be purchased in-app; the relaunched plans “meter usage in credits and introduce a weekly usage limit,” with Pro/Max at “6× and 14× the Lite usage allowance.”
- **Use API Key** — manual Base URL + key per account type: “For a Coding Plan, the OpenAI URL must be the Coding-only endpoint (`/api/coding/paas/v4`), not the general endpoint `/api/paas/v4`.” Coding endpoints: `https://open.bigmodel.cn/api/coding/paas/v4` / `https://api.z.ai/api/coding/paas/v4`; Anthropic endpoints: `https://open.bigmodel.cn/api/anthropic` / `https://api.z.ai/api/anthropic`; general OpenAI endpoints: `…/api/paas/v4`. “The Anthropic Base URL does not apply to resource packages / prepaid balance” — it bills only for accounts that never bought a plan plus allowlisting; use the general OpenAI endpoint instead.
- **Team plans** — “the connection menu on the BigModel provider page lists each team you have access to as its own Team Plan entry”; “seat management happens on BigModel's team plan page, not in the client.”

Third-party providers (Add Provider): Anthropic (`https://api.anthropic.com`), OpenRouter (`https://openrouter.ai/api`), Moonshot (`https://api.moonshot.cn/anthropic`), OpenAI (`https://api.openai.com`), and DeepSeek-compatible endpoints (“Set the Anthropic endpoint to `https://api.deepseek.com/anthropic`” and “Set the OpenAI endpoint to `https://api.deepseek.com/v1`”). “The ZCode application itself is completely free. As a developer, you still need your own API Key or a model service plan.”

## Model settings semantics

- **Per-model advanced settings** — “Max output tokens” per model; “Leave it empty and ZCode uses whatever the model itself supports … output length and conversation history compete for the same space, so a higher limit means ZCode compacts long conversations sooner.”
- **Context windows** — “Custom-provider models: editable … Coding Plan / Start Plan built-in models: the context window is issued by the server. Local edits are restored to the official value on the next sync … Models whose name ends in [1m]: fixed at a 1M context, not editable.” “A newly added model, or one without window metadata, defaults to 200,000.” “Built-in GLM models connected through a Coding Plan use a fixed window that can't be edited.”
- **Compaction** — “Automatic context compaction triggers before the window is actually full: the system reserves room for model output plus a safety buffer (about 34K tokens combined) … a 128K window compacts around 94K tokens, a 1M window around 966K. There is currently no user-facing switch or threshold for automatic compaction.”
- **Thinking effort** — GLM-5.3 offers “low / high / max” with “max” default; GPT series low→xhigh; Claude series low→xhigh (Opus 4.7 adds max); Kimi K3 and DeepSeek V4 low/high or high/max. “OpenAI-compatible endpoints use the top-level `reasoning_effort` field, while Anthropic-compatible endpoints use `thinking` / `effort`.” Caveats: “a third-party deployment of the same model may accept different levels than the official endpoint”; provider-specific toggles (“`enable_thinking` for the Qwen family”) are supported, but “GLM's clear_thinking and MiniMax's adaptive thinking mode cannot currently be configured in ZCode,” and “in `~/.zcode/v2/config.json`, a provider's options only recognizes connection settings such as `apiKey`, `baseURL`, `apiKeyRequired`, and `headers`; any other keys added by hand … are silently ignored and never written into the request body.”
- **Image support** — decided per provider/model config with three states: “Supported … Unsupported: the image data is removed before the request is sent and replaced with a text notice … Unknown: ZCode does not block the image.”

## Official sources

- [Welcome](https://zcode.z.ai/en/docs/welcome) · [Install](https://zcode.z.ai/en/docs/install) · [Connect Models / configuration](https://zcode.z.ai/en/docs/configuration) · [Changelog](https://zcode.z.ai/en/changelog) · [FAQ (Q&A)](https://zcode.z.ai/en/docs/qa)
