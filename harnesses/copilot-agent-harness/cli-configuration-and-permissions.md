# CLI configuration and permissions

**Verified:** 2026-10-29 · **Surface:** Copilot CLI

## Configuration home and trust

Copilot CLI uses `~/.copilot/` by default. Setting `COPILOT_HOME` relocates its configuration home, including user instructions and hooks.

On launch in a directory, the CLI asks whether to trust files in and below that directory. Trust can apply only to the current session or be saved for future sessions. Persistent entries live in the automatically managed `config.json`:

```text
~/.copilot/config.json
```

The `trustedFolders` array contains the saved directories. Trust is security-sensitive: project MCP files are silently skipped in untrusted directories.

## Tool approval model

Interactive `/permissions manual` mode asks before unapproved tool use. `/permissions assisted` lets a model assess eligible requests while preserving existing permissions and enterprise rules.

Useful command-line controls:

| Option | Effect |
|---|---|
| `--allow-tool='shell'` | Allow a tool or shell class without individual approval |
| `--allow-tool='shell(git status)'` | Allow a specific command form |
| `--deny-tool='shell(git push)'` | Deny a command or tool |
| `--allow-all-tools` | Bypass ordinary tool-approval prompts |
| `--available-tools=...` | Limit which tools are available at all |

A deny rule takes precedence over `--allow-all-tools` and `--allow-tool`. Enterprise restrictions also remain effective. For MCP tools, use `SERVER(tool)` to target one tool or `SERVER` to target the server.

“Approve this tool for the session” is broader than approving one invocation. For example, approving `rm` for the session can permit later `rm` commands with different arguments.

## Path permissions

Default path access covers:

- the current working directory;
- its descendants;
- the system temporary directory.

Relevant controls:

- `--allow-all-paths` disables path verification.
- `--disallow-temp-dir` removes the default temporary-directory grant.

Path checks apply to shell commands, file operations, and search tools, but shell path extraction is heuristic. Complex constructs, custom environment variables, and not-yet-created symlink targets can evade reliable detection. Path approval is therefore not equivalent to an operating-system security boundary.

## URL permissions

URLs require approval by default. Controls include:

- `--allow-url=DOMAIN`;
- `--deny-url=DOMAIN`;
- `--allow-all-urls`.

Checks cover `web_fetch` and selected network-capable shell commands. URL extraction is also heuristic: URLs hidden in files, configuration, environment variables, or obfuscated strings may not be detected. HTTP and HTTPS are distinct approval targets.

## Blanket mode

`--allow-all` and `--yolo` combine:

```text
--allow-all-tools + --allow-all-paths + --allow-all-urls
```

Interactive equivalents are `/permissions allow-all`, `/allow-all`, and `/yolo`. This is a high-risk convenience mode, not an override of explicit deny rules or administrator policy.

## Sandbox boundaries

Local command sandboxing is off by default. It can reduce the effect of broad grants, but it is separate from running the whole session in a cloud sandbox. Do not describe a cloud-hosted run as proof that local CLI commands are sandboxed, or vice versa.

## Telemetry scope

`--disable-client-telemetry` disables first-party client-side telemetry for that invocation only. It does not disable GitHub server-side session collection, explicitly configured OpenTelemetry exports, or telemetry forwarding explicitly enabled by an SDK application.

## Sources

- [Configuring Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/configure-copilot-cli)
- [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [Adding MCP servers](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
