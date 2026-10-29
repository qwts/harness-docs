# Configuration, profiles, and providers

**Verified:** 2026-10-29

## Harness and agent homes

`$DSH_HOME` defaults to `~/.dsh`. `$DSH_AGENTS_HOME` defaults to `~/.agents` and supplies shared instructions and skills.

Important Harness-home entries include:

```text
$DSH_HOME/
  AGENTS.md
  cordis.patch.yml
  .credentials.yaml
  .env
  profiles/
    <name>/
      package.json
      cordis.patch.yml
  skills/
  sessions/
  logs/
```

Use an explicit, isolated `$DSH_HOME` for automation or untrusted workloads. The Python SDK guide recommends isolated workspace and Harness-home paths and its example does not silently read `~/.dsh`.

## Profiles are composed plugin trees

A profile is not merely a model preset. Its `package.json` contains out-of-tree dependencies and the `dsh.profile` manifest with an ordered bundle list. Its `cordis.patch.yml` supplies the profile’s own patch layer.

The tree is composed over an empty root in this order:

1. each bundle patch in `dsh.profile.bundles` order;
2. the profile’s `cordis.patch.yml`;
3. the home-level `$DSH_HOME/cordis.patch.yml`;
4. each `--patch <path>` overlay in argument order.

Later layers win per row. **The home patch outranks the profile patch.** A patch replaces a targeted row’s complete `config` value rather than deep-merging individual keys, so an override must preserve every field the row still needs.

`web`, `headless`, `sdk`, `sdk-minimal`, and `acp` auto-initialize from shipped templates. The `desktop` name is reserved for the Electron-owned profile; the ordinary npm CLI rejects attempts to boot it.

Inspect composition without booting:

```sh
dsh --profile web --dump-default-config
dsh --profile web --patch ./extra.yml --dump-config
dsh --profile web --dump-config-schema
```

`--dump-default-config` shows bundle layers. `--dump-config` adds profile, home, and CLI overlays. `--dump-config-schema` imports plugin declarations to produce JSON Schema; review that operation’s safety and scope before using it with untrusted plugins. Dump output and startup diagnostics can contain sensitive values and are not automatically safe to share.

## Reload boundaries

Configuration HMR, when enabled, watches the profile manifest and profile/home patches and recomposes them through a serialized reload. Without HMR, edits apply on restart.

Bundle membership is different: after adding, removing, or updating a bundle with `dsh plugin`, restart the profile. A running profile retains its current bundle set. Ordinary patch edits can hot-reload where HMR is enabled.

## Model providers

The Web UI configures providers under **Settings → Models**. Model changes take effect on the next request without restarting the server.

Custom model APIs require:

- a permanent lowercase provider ID;
- base URL;
- one API protocol;
- credentials or an environment fallback;
- at least one model.

Supported protocol values are:

```text
openai-completions
openai-responses
anthropic-messages
```

One provider speaks one protocol. A gateway exposing multiple protocols needs separate provider entries. Selecting a model also makes it the default for new sessions; an existing session that has sent a request retains the model recorded in its log.

Advanced provider settings live in the active profile’s `cordis.patch.yml`. For ordinary Web launch, that is `$DSH_HOME/profiles/web/cordis.patch.yml`.

## Credential handling

Saved keys are write-only in the Models UI and are stored in `$DSH_HOME/.credentials.yaml`; settings retain credential references rather than the literal key.

Base-backed profiles resolve provider credentials in this documented order:

1. inherited environment;
2. `$DSH_HOME/.credentials.yaml`;
3. the invoking directory’s `.env`;
4. `$DSH_HOME/.env`.

The managed credential document is not materialized into `process.env`. The two `.env` files are ordinary launch-environment layers. Do not commit credentials, and do not assume a credential visible in the UI has been validated until a real request succeeds.

## Logs and telemetry are separate from local UI hosting

A local Web UI or Desktop shell does not imply that all data remains local.

Base profiles enable the DeepSeek session-log contributor by default. It sends complete unaccepted log suffixes with subsequent DeepSeek requests, including requests through configured gateways. These exports can include message text, tool arguments and results, and workspace paths.

Disable it through **Settings → General → Upload Session Log when using the official model API**, or in a profile patch:

```yaml
- name: '@deepseek-ai/dsh-session-log-deepseek'
  config:
    enabled: false
```

OTel session upload is a separate, feedback-gated path. `DSH_TELEMETRY_MODE=DISABLED` disables OTel delivery, and a nonempty `DSH_TELEMETRY_DISABLED` disables its row. Those OTel controls do **not** enable or disable the DeepSeek session-log contribution.

Review both paths when evaluating privacy. Also review configured model providers, MCP servers, plugins, web tools, and network proxies: each can create its own data path.

## Sources

- [CLI behavior reference](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/reference/README.md)
- [CLI launcher](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/README.md)
- [Configure models](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md)
- [Python SDK guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/python-sdk.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
