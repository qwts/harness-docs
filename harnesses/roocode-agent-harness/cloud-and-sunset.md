# Roo Code Cloud, Router, and the sunset

**Verified:** 2026-11-16 (final state; all Roo Code products shut down May 15, 2026)

## What Roo Code Cloud was

“Roo Code Cloud is the web platform companion to the Roo Code VS Code extension. It extends the extension's functionality (task syncing and sharing, usage analytics) and allows you to run tasks in the cloud, completely independent from the extension, via Cloud Agents.” Philosophy: “All of it follows our philosophy of being model-agnostic and letting you bring your provider key to stay in control of inference models and costs.”

| Feature | Availability |
|---|---|
| “Connect the Extension to Cloud” (GitHub, Google, or email) | Free |
| Analytics — “Track model tokens, tasks, estimated cost, and Cloud Agent credits” | Free |
| Task Sync — “Real-time task updates and cloud task history across devices” | Free |
| Task Sharing — “Share tasks via secure, expiring links” | Free |
| Cloud Agents — “Run Roo in the cloud with credit-based runtime and BYO provider keys” | Paid |

**Cloud Agents** — “specialized, autonomous AI workers that run in the cloud … available in all Roo Code Cloud plans, including the free one.” The team: **the Explainer** (“Technical Educator”), **the Planner** (“Implementation Planning and Architecture” — “take a high-level idea or a Product Requirement Document (PRD) and break it down into a thorough, step-by-step implementation plan”), **the Coder** (“writes code, creates pull requests, and implements features”), **the PR Reviewer** (code reviews), and **the PR Fixer**. Agents are renamable. Working style: “they work on isolated branches and deliver work via PRs.”

**GitHub integration** — via the Roo Code GitHub App: “the Coder agent delivers its work via git … push to the remote repository and create a PR”; “the Reviewer agent delivers its work via Github PR comments … It will automatically review newly created PRs (including by the Coder!)”; “the Fixer agent delivers its work via pushes to Github branches (and comments)” when invoked from a PR. A Slack integration brings agents into channels.

**Preview environments** — “let you run multiple services in a cloud container with automatic domain routing. Each port you configure gets a unique public URL, and environment variables are injected” (`ROO_WEB_HOST`, `ROO_API_HOST`); YAML-configured repositories/commands with detached services.

**Team plan** — “Centralized LLM inference billing without managing API keys (with Roo Code Router) … Shared Cloud Agent team (with centralized billing) … Centralized integration with Github and Slack (not one per user) … Extension configuration enforcement (providers/models, MCP servers and more).” 14-day trial entry point.

**Credits** — “Credits are just pre-paid dollar amounts you can optionally buy to use the paid parts of the Roo Code suite.” “**Roo Code VS Code** (the extension) is 100% free”; “**Roo Code Router** (the model provider) offers both free and paid models”; “**Roo Code Cloud Agents** … are a paid feature.”

## Roo Code Router

“we've built the Roo Code Router … it only needs your Roo Code Cloud account and offers a selection of models we tested and approved to work with Roo Code” — frontier-lab models plus “Tested open weight models from upcoming labs (Moonshot AI, MiniMax, GLM, etc).” Privacy: “We never use your data for training and don't keep logs of the prompts themselves,” with the task-sync/Cloud-Agent retention caveat and the stealth-model training caveat documented.

## The sunset

“All Roo Code products will be shut down on May 15, 2026. This includes the Roo Code VS Code Extension, Roo Code Cloud, and Roo Code Router.” Rationale: “we don't believe IDEs are the future of coding.” What happened on the date: Cloud and Router shut down, “Refund any unused balances,” and the extension repository archived (the README adds “The Roo Code Extension was shut down on May 15th”).

Recommended successors: “If you want a model-agnostic open-source extension, we recommend Cline”; “check out our new home at roomote.dev” (cloud agents); the README also names “ZooCode (a fork started by the Roo Code community)” and repeats the Cline recommendation (“from where Roo Code originated”). Billing contact after sunset: billing@roocode.com. Milestone noted in the farewell: “pushed us past 3 million extension downloads.”

## Official sources

- [What is Roo Code Cloud?](https://roocodeinc.github.io/Roo-Code/roo-code-cloud/what-is-roo-code-cloud) · [Cloud Agents](https://roocodeinc.github.io/Roo-Code/roo-code-cloud/cloud-agents) · [GitHub Integration](https://roocodeinc.github.io/Roo-Code/roo-code-cloud/github-integration) · [Preview Environments](https://roocodeinc.github.io/Roo-Code/roo-code-cloud/environments) · [Team Plan](https://roocodeinc.github.io/Roo-Code/roo-code-cloud/team-plan) · [Router Overview](https://roocodeinc.github.io/Roo-Code/roo-code-router/overview) · [Credits](https://roocodeinc.github.io/Roo-Code/credits/overview) · [Sunsetting Roo Code](https://roocodeinc.github.io/Roo-Code/sunset)
