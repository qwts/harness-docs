# Meta Muse Code corrections log

**Verified:** 2026-10-29

| Claim | Verdict | Correction |
|---|---|---|
| The current executable is `muse-code`. | ❌ FALSE | Current official documentation uses `muse`. |
| `MUSE_CODE.md` is the project instruction file. | ❌ FALSE | Current documentation selects among `AGENTS.md`, `CLAUDE.md`, `.agents/AGENTS.md`, and `.claude/CLAUDE.md`. |
| Muse Code supports only macOS and Linux. | ❌ FALSE | Windows is also supported. |
| User settings are stored at `~/.muse/settings.json`. | ❌ FALSE | The documented path is `~/.config/muse/settings.json`. |
| An organization's `~/.muse/worktrees` territory is the vendor settings path. | ❌ FALSE | That is organization-specific operational metadata, not Muse Code's documented user-settings location. |
| `muse init` creates settings and several agent configuration files. | ❌ FALSE | It writes only `AGENTS.md` and does not change `settings.json`. |
| Project instructions load before workspace trust. | ❌ FALSE | They load only after trust. |
| Muse concatenates every supported instruction-file alternative in a directory. | ❌ FALSE | It uses a documented per-directory selection order. |
| Shallower project rules override deeper ones. | ❌ FALSE | Deeper project rules take priority; project rules also beat user rules. |
| New sessions default to Unrestricted. | ❌ FALSE | The default new-session profile is Auto-review. |
| Rejecting one stage of a compound command only skips that stage. | ❌ FALSE | Rejection prevents the entire compound command from running. |
| The sandbox is opt-in and fails open. | ❌ FALSE | It is on by default and fails closed. |
| `.git`, `.muse`, and `.agents` are writable whenever the workspace is writable. | ❌ FALSE | They remain read-only inside the writable workspace. |
| `--disable-sandbox` changes command execution only. | ❌ FALSE | It also removes file-tool workspace confinement and forces full network egress. |
| Every subagent is worktree-isolated by default. | ❌ FALSE | Children share the lead checkout by default; isolation is selected per child. |
| `--subagent-worktree-isolation` isolates every child. | ❌ FALSE | It is a compatibility flag and does not impose universal isolation. |
| Muse silently uses a shared checkout if worktree creation fails. | ❌ FALSE | It does not silently fall back from requested isolation. |
| Default capacity is eight children plus the root. | ❌ FALSE | The default tree capacity is eight including the root. |
| `muse exec` exit 0 means tests and the task succeeded. | ❌ FALSE | It means the turn completed; CI needs explicit validation. |
| Workflows are guaranteed in every build. | ❌ FALSE | Availability is rollout/build dependent. |
