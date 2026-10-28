# CLI, CI, and SDK

**Verified:** 2026-10-29

## Command surfaces

| Surface | Command or package |
|---|---|
| Interactive CLI | `muse` |
| Headless execution | `muse exec` |
| Programmatic server | `muse serve` |
| Schema inspection | `muse schema` |
| SDK | `@muse-code/sdk` |

Muse Code supports macOS, Linux, and Windows. Current documentation, not launch-era third-party summaries, is authoritative for the executable name and platform list.

## CI authentication

Headless CI authentication uses:

```text
META_API_KEY
```

Treat this as a secret in the CI provider. Do not place it in `AGENTS.md`, project memory, workflow source, or committed settings.

## Exit-code semantics

For `muse exec`, exit code `0` means the turn completed. It does not prove that:

- the requested task was completed correctly;
- repository tests passed;
- generated code was accepted;
- every tool call succeeded.

CI must gate on explicit validation. A robust job should run tests or other task-specific checks and fail on their results rather than using the agent process exit code as the quality signal.

## Selecting an integration surface

- Use interactive `muse` for a person supervising a session.
- Use `muse exec` for one-shot automation with external validation.
- Use `muse serve` when a client needs a server interface.
- Use `muse schema` to inspect machine-readable interface structure.
- Use `@muse-code/sdk` for application integration instead of scraping interactive terminal output.

Permissions, trust, and sandbox behavior remain relevant in automation. A headless runner does not become safe merely because no interactive approval UI is present.

## Sources

- [Muse Code overview](https://dev.meta.ai/docs/muse-code)
- [Extending Muse Code — Headless and CI](https://dev.meta.ai/docs/muse-code/extending)
- [Models](https://dev.meta.ai/docs/models)
