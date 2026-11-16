# Sessions, Goal Mode, and agent features

**Verified:** 2026-11-16 (ZCode docs zcode.z.ai/en/docs; app v3.18.2)

## The ZCode Agent workspace

“ZCode Agent is the default self-developed agent in ZCode and the primary entry point when creating a new task.” It is “deeply adapted for the GLM-5.3 model family” and “performs especially well on complex project understanding, long-task planning, multi-turn context retention, and continuous code changes.” The model side: “Stable 1M context,” “Flexible effort … surfaced in ZCode as the thought level toggle,” and “RL trained for agentic coding … with anti-hack mechanisms so the model optimizes for real engineering work rather than surface rewards.”

“With ZCode Agent, you can @ reference files in the input box, use / commands, call $ skills, and switch models, execution modes, and branches inside the same task.” A **Git branch picker** is part of the workspace entry. **Working without a project**: “choose Work outside a project in the workspace menu … On first launch with no session to restore, ZCode starts here by default.” Files created in such conversations “stay on disk and are not cleaned up when you close the conversation,” the conversation “is fixed in this mode” after the first message, and “The Git branch picker is hidden in this mode too.”

**Thought level**: “higher levels think more thoroughly and are usually more reliable, at the cost of slower responses,” switched via “the thinking icon next to the model name”; “For GLM-5.3 there are three levels, with Max as the default” (Low “prioritize response speed,” High “balancing speed and quality,” Max “the deepest level (default)”).

## Goal Mode

“For complex work that runs for a long time, use `/goal` to set an objective for the current session. Once it is set, ZCode Agent keeps working toward that objective: at the end of every round it automatically checks whether the goal has been met, starts another round if it hasn't, and only wraps up once it has.” Forms: `/goal`, `/goal <objective>`, `/goal replace <objective>`, `/goal pause`, `/goal resume`, `/goal clear`. “A session has one goal at a time.” The summary panel shows “the objective and the elapsed time,” finished checklist items “grouped by iteration,” and “Each round's title comes from the next action the previous round's verification produced.”

## Side conversations and forking

A **side conversation** is “a separate tab in the right-hand panel that runs alongside the main conversation without interfering with it.” Open it from a selection (“Ask in side conversation”), via “`/side` or `/btw`” (“Both commands do the same thing and are not sent as messages”), the + menu, or the start screen. “A side conversation uses the main session's history as model context … It's a fully capable conversation: it can call tools, edit files, and go through permission confirmations.” Limits: “Not available — Goal mode, editing a sent message, forking” are “Goal mode, editing a sent message, forking”; “One side conversation per task at a time” per window; “Platform — Desktop only”; and “closing the tab, deleting or archiving the parent task, or quitting the app closes it … and it can't be recovered.”

**Forking**: “Hover over an assistant message that has finished answering and click Fork … ZCode creates a new task starting from that message, titled Fork of <original title> … The new session inherits the history up to the fork point, the model and thought level in effect at the time, and any goal-mode progress. The original session is preserved and continues to run unchanged.” Constraints: “You can only fork from a cleanly completed assistant message”; “Forking only affects conversation history — it does not roll back files on disk. Both sessions operate on the same workspace”; “The queued messages, in-flight work, and background tasks aren't copied”; “Forking isn't available while a compaction is running.”

## Browser automation

“ZCode has a built-in browser panel for opening local dev servers or live pages inside the app … the agent can drive it — you describe what you want in one sentence, and it opens URLs, clicks, fills forms, scrolls, takes screenshots, and decides what to do next based on what's actually happening on the page.” “This comes from the official Browser Use plugin and is on by default; if it has been turned off, re-enable it under Settings → Browser → Enable built-in browser control, and it applies to new sessions.” “The browser panel is desktop-only,” handles “http, https, and file URLs,” has “an address bar plus back, forward, reload, and DevTools buttons,” an element picker (“Turn page elements into context”), and per v3.18.2 “device size presets, and pages can be exported as PNG or PDF.” “The plugin also brings two skills: one teaches the agent how to drive the browser, the other is a walkthrough-style testing flow.”

## Worktrees and parallel work

The v3.18.2 changelog: “Added support for Worktree for parallel task development.” The docs do not yet carry a dedicated Worktree topic page — the feature is release-noted only (documented as far as the docs go).

## Other desktop features

- **Edit History** — revise the last user turn (“only the user message of the last turn can be edited”).
- **Repo Wiki** — generated architecture guide with per-claim source locations; one per workspace, stored in the workspace's runtime environment.
- **Usage Stats** — per-connection usage; the sidebar “quota shown … follows the selected connection, and quota cards indicate whether they're personal or team.” The v3.18.2 changelog notes “Usage statistics records are no longer automatically cleared upon expiration.”
- **Shares** — v3.18.2 adds “setting an expiration period, and a new share management page.”
- **Voice input** — v3.18.2: “The input box now supports voice input, with speech transcribed to text in real time.”
- **Session management Tool** — v3.18.2: “Added a Session management Tool for enhanced self-management.”

## Keyboard shortcuts

A dedicated shortcuts page covers the desktop defaults; the two recurring bindings in the docs are Shift+Tab (cycle execution modes) and Command/⌘+K (Command Center).

## Official sources

- [ZCode Agent](https://zcode.z.ai/en/docs/agents) · [Goal Mode](https://zcode.z.ai/en/docs/goal) · [Browser Automation](https://zcode.z.ai/en/docs/browser-use) · [Subagents](https://zcode.z.ai/en/docs/subagents) · [Edit History](https://zcode.z.ai/en/docs/edit-history) · [ADE Tools](https://zcode.z.ai/en/docs/ADE-tools) · [Keyboard Shortcuts](https://zcode.z.ai/en/docs/keyboard-shortcuts) · [Changelog](https://zcode.z.ai/en/changelog)
