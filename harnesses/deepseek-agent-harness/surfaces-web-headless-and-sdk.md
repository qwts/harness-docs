# Web, headless, and SDK surfaces

**Verified:** 2026-10-29

## Surface model

`dsh` is the sole supported Node application launcher. ACP and SDK are profiles, not separate public binaries.

| Surface | Invocation | Behavior |
|---|---|---|
| Web | `dsh web` | Browser UI backed by the Web profile |
| Desktop | Electron application | Native shell around the complete Web application |
| Headless | `dsh --profile headless "task"` | One persisted task, final answer, then exit |
| ACP | `dsh --profile acp` | Agent Client Protocol over stdio |
| SDK | `dsh --profile sdk` | Full SDK runtime over stdio JSON-RPC |
| Minimal SDK | `dsh --profile sdk-minimal` | Standalone minimal agent tree over stdio JSON-RPC |

The invoking directory is the default workspace root for base-backed modes. The `web`, `headless`, `sdk`, `sdk-minimal`, and `acp` profiles auto-initialize on first use.

The generated help includes a `dsh tui` example only as an assumed installed terminal profile. Do not present it as evidence of a shipped default interactive TUI.

## Web

The documented npm launch is:

```sh
npx @deepseek-ai/dsh web
```

It defaults to `http://127.0.0.1:3080` and opens a browser for a local launch. SSH launch prints the host URL without opening a browser; `--no-open` disables browser opening explicitly.

Useful server flags include `--host`, `--port`, `--no-open`, `--public-url`, `--trusted-host`, `--tls-cert`, and `--tls-key`. Wildcard bind addresses are rejected; bind one concrete local interface address.

### Public deployment boundaries

`--public-url` advertises the browser-facing URL; it grants no trust. A proxy authority must also be admitted with `--trusted-host`. The launch token in the printed URL is a process credential, and the signed session cookie authenticates browser access. TLS protects a network leg but does not authenticate the browser or replace the Host/Origin fence.

Restrict the listening port to the trusted proxy or network. Do not expose the default listener or printed token publicly and assume `--public-url` provides access control.

## Desktop

Desktop is an Electron shell around the complete Web application, using a shared profile runner and authenticated Web Host. It shares sessions, settings, credentials, workspaces, and storage under `$DSH_HOME` with CLI `dsh`, while executable packages, plugin activation, and lockfiles remain separate.

The `desktop` profile is reserved for Electron. The ordinary npm CLI rejects booting it; a Desktop-installed `dsh` command can manage the initialized Desktop profile through that installation’s runtime.

Closing the main window ordinarily hides it while the page and Host continue running. Quit is a separate operation that can interrupt active work or prevent scheduled reminders from running.

## Headless operation

```sh
dsh --profile headless "run the tests"
echo "run the tests" | dsh --profile headless
dsh --profile headless --json "run the tests"
dsh --profile headless --session-id session-… "continue"
```

By default, a one-shot run creates a fresh persisted session, prints the final nonempty assistant text to stdout, and exits. Provider reasoning deltas go to stderr under a `dsh: reasoning:` heading. `--json` replaces final-message stdout with newline-delimited run events.

Exit status is `0` only when the final turn reason is `completed`; otherwise it is `1`. This is more specific than “process produced an answer,” but it still does not prove tests passed or the task was performed correctly. Run explicit validation.

Automation controls include:

- `--permission-mode <preset>`;
- `--approval-policy ask|never`;
- repeatable `--add-dir <absolute-path>` for additional `workspace-write` roots;
- `--session-id <id>` for an existing persisted session.

The headless profile opens no HTTP listener. Its base-backed permission defaults differ from `sdk-minimal`, as described below.

## ACP

`dsh --profile acp` serves automation clients over ACP stdio until disconnect. ACP is a protocol surface, not a promise that every client exposes the same permission controls, UI, or continuation behavior as Web.

## Python SDK and JSON-RPC

Install the published package with Python 3.10 or newer:

```sh
python -m pip install deepseek-harness-sdk
```

The wheel includes a matching native runtime and `dsh`, so normal SDK execution needs no system Node.js. The SDK launches its selected stdio JSON-RPC profile lazily and reuses it until the context manager exits.

```python
from deepseek_harness import DeepSeekHarness

with DeepSeekHarness(
    provider="deepseek-official",
    model="deepseek-v4-flash",
    cwd="/absolute/path/to/disposable-workspace",
    dsh_home="/absolute/path/to/isolated-dsh-home",
    profile="sdk-minimal",
) as harness:
    result = harness.run("Inspect the repository.", session_id="example-001")

print(result.final_response)
```

Use explicit workspace and Harness-home paths. The SDK example and SDK do not silently read `~/.dsh`.

### `sdk-minimal` is not a smaller safe default

The standalone minimal tree intentionally differs from base-backed `sdk`:

- it pins `danger-full-access` and has no approval or permission-settings service;
- its platform-selected persistent shell can modify any path visible to the runtime;
- it provides the shell, `read_image`, and `working_directory` by default;
- it omits text filesystem tools, local instruction discovery, subagents, Web tools, managed settings/credentials, and SQLite;
- it stores uncompressed JSONL sessions under `<dsh_home>/sessions`;
- it enables DeepSeek session-log upload by default unless disabled in a patch.

Use a disposable checkout or container. Adding `str_replace_editor` through a patch does not add workspace confinement; its local filesystem provider can access paths outside the runtime working directory.

The installed wheel also packages the full Web profile, but Web and Python SDK serving are separate applications. One cannot substitute for the other’s client protocol.

## Sources

- [Project README](https://github.com/deepseek-ai/deepseek-harness/blob/master/README.md)
- [Generated CLI help](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cli-help.md)
- [CLI behavior reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
- [Web UI guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/index.md)
- [Public deployments](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/public-deployments.md)
- [Desktop reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.md)
- [Python SDK guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/python-sdk.md)
