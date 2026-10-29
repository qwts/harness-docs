# MCP, hooks, plugins, and integrations

**Verified:** 2026-10-29

## Extensions are MCP capabilities with several lifecycles

Goose extensions use MCP and can be built-in, platform, local stdio, or remote Streamable HTTP servers. Built-in goose extensions are MCP servers in their own right and can also be used by other agents. Deprecated SSE transport is not supported.

Configuration in `config.yaml` sets defaults for future sessions. Mid-session extension changes preserve the current conversation but affect only that session; they do not rewrite defaults. CLI `--with-extension`, `--with-builtin`, and `--with-streamable-http-extension` similarly add extensions for the launched session without installing them as persistent defaults.

The Extension Manager can discover, enable, and disable extensions during a session. Dynamically enabled extensions remain session-scoped. Official examples show a CLI confirmation before dynamic enablement; do not infer a universal approval guarantee for every interface and permission configuration.

Goose checks external extensions for known malware before activation and can block detected malicious packages. This is a known-threat check, not a provenance guarantee or behavioral audit.

## Installation allowlist

`GOOSE_ALLOWLIST` names a URL containing YAML extension IDs and exact command strings. Goose fetches and caches it when first needed and refetches on restart. Installation is rejected when the command does not match.

The allowlist:

- applies to CLI and Desktop extension installation;
- requires exact command matching;
- does not constrain what an approved server’s tools do;
- is not an OS sandbox or a complete defense against direct configuration changes;
- should use HTTPS and restricted edit access.

## Remote extension authentication

Streamable HTTP extensions can use OAuth through Client ID Metadata Documents or Dynamic Client Registration. Servers requiring out-of-band registration can set `client_id` and optionally `client_secret_key`; the latter references secret storage rather than embedding the secret. Explicit scopes can narrow the request instead of accepting potentially broader advertised defaults.

The OAuth callback normally listens on `127.0.0.1` with an ephemeral port. `GOOSE_OAUTH_CALLBACK_PORT` supports authorization servers requiring a fixed preregistered redirect port.

## Code Mode changes orchestration, not tool authority

The optional Code Mode extension exposes three meta-tools: `list_functions`, `get_function_details`, and `execute_javascript`. The model writes JavaScript to discover tools, batch calls, and chain results instead of receiving every tool definition on every model call.

Each execution uses a fresh goose-owned Boa worker/context. The JavaScript environment has no direct filesystem, shell, network, environment, or package access, and discards state between calls. However, `tools.call` still invokes enabled extension tools under goose’s extension lease, permission flow, cancellation, and mutation handling. The restricted JavaScript context therefore does **not** sandbox the tools it calls.

Only assistant-visible text results reach the script; images, binary data, and other content types are ignored. Execution and output limits apply, and timeout/cancellation stops in-flight calls.

## MCP Apps are sandboxed UI resources

The Apps extension creates single-file HTML/CSS/JavaScript apps exposed as MCP App resources in standalone sandboxed windows. They have no external dependencies or npm packages and are stored as HTML files in goose’s data directory.

That window sandbox belongs to the custom app UI. It does not confine the agent, Developer shell, MCP servers, or other tools that created the app.

## Plugins package skills and executable hooks

Current goose plugins provide skills, hooks, or both; they are not simply another name for MCP extensions.

Recommended locations:

- User: `~/.agents/plugins/<plugin-name>/`
- Project: `<project>/.agents/plugins/<plugin-name>/`

`goose plugin install <git-url>` clones, detects the format, copies the plugin into the user plugin directory, and reports imported components. Optional auto-update checks before plugin skills load, is rate-limited, and keeps the installed copy on update failure. Manual update replaces the installed copy from the original Git source.

Supported formats are Open Plugins and Gemini extensions. Open Plugins can carry `plugin.json`, skills, and `hooks/hooks.json`; Gemini extensions contribute skills. Open Plugin skills are namespaced; Gemini-extension skills are not.

Disable plugins through `disabledPlugins` in:

- `~/.config/goose/settings.json`;
- `<project>/.config/goose/settings.json`;
- `<project>/.config/goose/settings.local.json` for local-only project settings.

A disabled plugin is skipped during discovery, so its skills do not load and hooks do not run. Installing or auto-updating a plugin can change both model instructions and local commands; review the complete package and update provenance.

## Hook execution model

Hooks are plugin-owned shell commands configured in `hooks/hooks.json`. Goose runs command hooks with `sh -c`, sends a JSON event payload on stdin, sets `PLUGIN_ROOT`, and applies an optional timeout (30 seconds by default). Matchers are unanchored regular expressions, not globs; an invalid regex skips that rule with a warning.

Events cover session start/end, prompt submission, turn stop, tool pre/post/failure, file read/edit, and shell execution. Authority is narrow:

- `PreToolUse` can deny the tool call.
- `Stop` can force a turn to continue, subject to a consecutive-block cap.
- Other events—including `UserPromptSubmit`, `PostToolUse`, `BeforeReadFile`, `AfterFileEdit`, `BeforeShellExecution`, and `AfterShellExecution`—cannot block the associated action.
- `PreToolUseResult` observes the resolved allow/deny outcome before execution or denial return; delivery is awaited but best-effort and not durable.

### Blocking protocol

For a blocking event, goose interprets:

1. Exit code `2` as block, with the reason from stderr.
2. `{"decision":"block","reason":"..."}` on stdout as block regardless of exit status.
3. Exit `0` with empty stdout or valid allow JSON as allow.
4. Other output, delivery failure, spawn failure, or timeout as no decision/failure.

Keep logs on stderr: stray stdout can invalidate the decision protocol.

### Failure behavior

By default, a failed `PreToolUse` hook is logged and the tool proceeds. Set `on_failure: block` on that action to deny when it cannot complete. Limits matter:

- `on_failure` applies only to `PreToolUse`; `Stop` remains fail-open.
- It does not override an explicit denial or invalidate a clean allow.
- It does not make malformed configuration or invalid matchers fail closed.
- A malformed selected configuration can skip the plugin’s whole `hooks.json`; an invalid matcher skips only that rule.
- Hook-supplied denial reasons are not bounded or redacted, so hooks must not include secrets.

Hooks are powerful local code and incomplete coverage is easy. Test allowed, denied, malformed-output, timeout, missing-binary, and configuration-error paths before treating one as required policy.

## Trust and review matrix

| Artifact | Primary risk | Review focus |
|---|---|---|
| Stdio MCP extension | Local package/command execution and tool access | Command, package version, environment, credentials, tool schemas |
| Remote MCP extension | Remote data/actions and OAuth scope | Endpoint, client, scopes, headers, returned instructions |
| Recipe | Extensions plus shell retry/cleanup automation | Full YAML/JSON, parameters, commands, schedule |
| Plugin | Instruction and hook supply chain | Git source, update policy, skills, scripts, hook events |
| Hook | Arbitrary local shell and policy gaps | Payload parsing, matcher, stdout/exit protocol, failure mode |
| MCP App | Active HTML/JavaScript UI | App source, resource interactions, window behavior |

## Official sources

- [Using extensions](https://goose-docs.ai/docs/getting-started/using-extensions)
- [Configuration files](https://goose-docs.ai/docs/guides/config-files)
- [Extension allowlist](https://goose-docs.ai/docs/guides/allowlist)
- [Extension Manager](https://goose-docs.ai/docs/mcp/extension-manager-mcp)
- [Code Mode](https://goose-docs.ai/docs/guides/managing-tools/code-mode)
- [Apps extension](https://goose-docs.ai/docs/mcp/apps-mcp)
- [Plugins](https://goose-docs.ai/docs/guides/context-engineering/plugins)
- [Hooks](https://goose-docs.ai/docs/guides/context-engineering/hooks)
- [Extension architecture](https://goose-docs.ai/docs/goose-architecture/extensions-design)
