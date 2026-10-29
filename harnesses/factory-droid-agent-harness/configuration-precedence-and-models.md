# Configuration, precedence, and models

**Verified:** 2026-10-29

## Settings locations

Factory uses `.factory/` folders with the same settings schema at multiple levels. The authored levels are:

| Level | Location | Owner |
|---|---|---|
| Org | Managed-settings endpoint, org `.factory/` bundle, or system-managed `settings.json` | Organization administrators |
| Folder | `<git-root>/.../<subfolder>/.factory/` | Maintainers of a repository subtree |
| Project | `<git-root>/.factory/` | Repository maintainers |
| User | `~/.factory/` | Individual developer |

A `.factory/` folder can also contain `hooks.json`, `mcp.json`, and `droids/`, `commands/`, `skills/`, and `output-styles/` directories. Those extension files have their own discovery and trust rules; their presence does not make every field part of ordinary settings precedence.

Personal CLI settings live at:

```text
macOS/Linux: ~/.factory/settings.json
Windows:     %USERPROFILE%\.factory\settings.json
```

`settings.local.json` can sit beside `settings.json` in a user or project `.factory/` folder. It overlays the corresponding `settings.json` at the same level and follows that level’s hierarchy position. Keep machine-specific or sensitive local values out of version control. `.droid.yaml` is legacy; current configuration uses the `.factory/` files.

## Two opposite precedence systems

At resolution time, Factory places the authored levels into this complete build order, highest priority first:

```text
Org
Runtime
Folder
Project
User
Dynamic
BuiltIn
```

That list is **not** a single “last value wins” rule for every setting.

| Setting class | Effective precedence | Consequence |
|---|---|---|
| Hard controls | Highest level present wins; Org is authoritative | Lower levels may extend accumulative policy but cannot weaken or remove higher-level controls |
| Session defaults | Runtime > Folder > Project > User > Org > Dynamic > BuiltIn | More-local preferences win, but only within organization hard-control ceilings |

Hard controls include model and MCP policy, permission rules, sandbox policy, managed hooks, strict plugin controls, subagent model policy, and scalar ceilings such as `maxAutonomyLevel`. Session defaults include the values inside `sessionDefaultSettings`, such as model, reasoning effort, interaction mode, and autonomy level.

For example, a user model preference can replace the organization’s fallback model, but it cannot select a model excluded by `modelPolicy`. Likewise, a runtime autonomy default cannot exceed an organization `maxAutonomyLevel`.

Merge behavior also depends on data type:

- scalar hard controls use the first applicable value;
- documented array controls can union across levels;
- named arrays and objects can lock higher-level entries or keys while allowing new lower-level additions;
- session-default scalars use the more-local value;
- permission rules have separate ID resolution followed by `block > ask > allow` decision precedence.

Do not replace a settings file with a hand-written merge assumption. Check the field’s class and merge semantics.

## System-managed organization policy

For policy that must apply before a user signs in, administrators can install:

```text
macOS:      /Library/Application Support/Factory/settings.json
Linux/WSL:  /etc/factory/settings.json
Windows:    C:\Program Files\Factory\settings.json
```

A present system-managed file is the authoritative organization-settings source on that machine. If it is absent or unreadable, Droid falls back to the ordinary organization-settings flow. If it is present but malformed or fails schema validation, organization settings resolve to an empty policy and the failure is logged; the broken file is not silently bypassed.

## Personal session defaults

A typical user settings file can select defaults without changing hard policy:

```json
{
  "sessionDefaultSettings": {
    "interactionMode": "auto",
    "autonomyLevel": "off"
  },
  "cloudSessionSync": true,
  "hooksDisabled": false,
  "blockOnMcpLoad": false,
  "subagentAutonomyLevel": "inherit"
}
```

Documented defaults include:

| Setting | Default | Meaning |
|---|---|---|
| `sessionDefaultSettings.interactionMode` | `auto` | New sessions start in Normal rather than Spec Mode |
| `sessionDefaultSettings.autonomyLevel` | `off` | New sessions begin with manual approvals outside policy-allowed reads |
| `cloudSessionSync` | `true` | CLI sessions mirror to Factory web |
| `hooksDisabled` | `false` | Hook execution is enabled |
| `blockOnMcpLoad` | `false` | A turn need not wait for every MCP server to finish loading |
| `subagentAutonomyLevel` | `inherit` | Task-launched subagents inherit parent autonomy, subject to organization caps |

`commandAllowlist`, `commandDenylist`, and `commandBlocklist` are deprecated but still respected. New command policy should use `permissionRules`; review coexistence before removing legacy entries.

Local CLI hosting does not imply local-only session storage. With `cloudSessionSync` enabled, CLI sessions are mirrored to Factory web. Model providers, connectors, MCP servers, hooks, and other integrations can create additional data paths.

## Models and BYOK

`model` selects a Factory-managed or configured custom model. `reasoningEffort` is model-dependent: a model accepts only the levels listed for that model, not every value shown in the general settings schema.

Personal custom models apply to CLI and desktop surfaces, not hosted web or mobile. A generic chat-completion entry can use an environment reference instead of embedding a secret:

```json
{
  "customModels": [
    {
      "model": "your-model-id",
      "displayName": "My Custom Model",
      "baseUrl": "https://api.provider.com/v1",
      "apiKey": "${PROVIDER_API_KEY}",
      "provider": "generic-chat-completion-api",
      "maxOutputTokens": 16384
    }
  ]
}
```

Personal API keys remain local. Do not place literal provider credentials in project settings or commit them.

### Credential helpers are executable configuration

`apiKeyHelper` runs a shell command to obtain credentials at request time. It is honored in user and organization-managed settings but stripped from project and folder settings, preventing an untrusted repository from supplying that executable helper through those levels.

Helper cache lifetime resolves through:

1. `FACTORY_API_KEY_HELPER_TTL_MS`;
2. `apiKeyHelperTtlMs`;
3. a five-minute default.

Organization-managed custom models cannot rely on a newly configured static `apiKey`; use an environment reference, an approved helper, a keyless endpoint, or the documented Bedrock path. Treat helpers as privileged code and review their command, environment, output handling, and credential lifetime.

## Sources

- [Droid CLI settings](https://docs.factory.com/droid-cli/settings)
- [Enterprise Controls and managed settings](https://docs.factory.com/enterprise/hierarchical-settings-and-org-control)
- [Custom Models (BYOK)](https://docs.factory.com/model-independence/byok)
- [Factory App settings](https://docs.factory.com/factory-app/settings)
- [Telemetry and privacy](https://docs.factory.com/enterprise/telemetry/privacy)
