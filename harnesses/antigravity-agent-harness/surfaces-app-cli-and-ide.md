# App, CLI, and IDE surfaces

**Verified:** 2026-10-29

## Shared agent, different host behavior

Antigravity App and CLI run the same agent and share features including projects, rules, skills, MCP, subagents, hooks, and plugins. This does not make their host configuration identical:

- App settings are organized into global and project scopes in the application.
- CLI preferences live in `~/.gemini/antigravity-cli/settings.json`.
- App permission presets and CLI mode/permission keys have different names and defaults.
- App offers standalone scratch-directory conversations, built-in browser/PDF side panels, VCS review, and automations.
- CLI offers terminal rendering controls, keybindings, status-line/title scripts, and headless/streaming interfaces.

## CLI interactive and headless

Interactive CLI uses `agy`. Headless mode uses:

```sh
agy -p "Prompt"
```

Aliases are `--print` and `--prompt`. Response text goes to stdout; diagnostics go to stderr. Output formats are:

- `text`;
- `json` — one completion envelope;
- `stream-json` — NDJSON `init`, `step_update`, and `result` events.

`--json-schema` constrains structured output. Streaming input supports multi-turn programmatic sessions when paired with stream-json output.

Headless mode uses cached credentials. In non-interactive CI without cached authentication, it exits with an authentication-required error instead of waiting for a login prompt.

## Headless permission behavior

There is no interactive approval prompt. A tool requiring unavailable approval is soft-denied by default:

- the run can continue;
- a notice goes to stderr;
- the process can exit 0.

Workspace reads/writes are auto-allowed in the documented headless behavior, while shell commands default to Ask and are soft-denied unless allowed. Prefer scoped `permissions.allow` entries. `--dangerously-skip-permissions` auto-approves all tools and should be reserved for fully trusted prompts and environments.

Exit 0 means a response was produced, not that the task or tests succeeded. JSON/stream-json consumers should also inspect `status` and `error`. Unknown pinned models fail nonzero rather than silently falling back. The default response timeout is five minutes and can be changed with `--print-timeout`.

## IDE boundary

The standalone Antigravity IDE and editor extensions have their own interface and feature documentation. Rules, skills, MCP, hooks, and plugins are documented as available across surfaces, but this reference does not infer identical settings paths, lifecycle timing, or UI capabilities for every IDE host.

❓ **UNVERIFIED:** Full App/CLI/IDE parity for every customization and permission behavior. Verify the target surface before deploying a shared configuration.

## Sources

- [Feature overview](https://antigravity.google/docs/cli/features/)
- [Settings](https://antigravity.google/docs/settings/)
- [CLI settings](https://antigravity.google/docs/cli/settings/)
- [Headless mode](https://antigravity.google/docs/cli/headless/)
- [IDE extensions](https://antigravity.google/docs/ide/extensions/)
