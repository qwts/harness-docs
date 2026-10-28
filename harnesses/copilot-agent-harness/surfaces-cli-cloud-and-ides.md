# CLI, cloud, and IDE surfaces

**Verified:** 2026-10-29

## Surface boundary

Copilot documentation covers CLI, cloud agent, code review, the Copilot app, and multiple IDEs. Shared names such as “custom agent,” “MCP,” or “instructions” do not establish identical discovery paths or runtime behavior.

| Capability | Copilot CLI | Copilot cloud agent |
|---|---|---|
| Project MCP | `.mcp.json`, `.github/mcp.json` | Repository MCP settings on GitHub; agent-profile MCP where supported |
| MCP capability | Local and remote tools | MCP tools; resources and prompts unsupported |
| MCP invocation | Local approval and permission model | Configured tools can be invoked autonomously without per-call approval |
| Remote MCP OAuth | Surface-specific support | Unsupported for repository-settings and custom-agent-profile cloud configuration |
| Sandbox | Local sandbox optional; cloud sandbox separate | Hosted cloud environment |
| Environment setup | Local machine and trusted project | `copilot-setup-steps.yml` workflow on default branch |

## Headless CLI

`copilot -p` / `--prompt` executes programmatically and exits. Project MCP still depends on trust: already-trusted directories load it automatically; untrusted directories skip it unless the documented prompt-mode environment override is set.

Custom agents can be selected with `--agent`. Headless use does not erase deny rules, enterprise restrictions, path heuristics, or the security consequences of blanket permission flags.

## Cloud-agent environment setup

The cloud agent can run a custom setup workflow:

```text
.github/workflows/copilot-setup-steps.yml
```

Requirements include:

- the file must exist on the repository's default branch;
- it contains one job named `copilot-setup-steps`;
- the job runs in the cloud agent's environment before the agent starts work.

The cloud firewall does not automatically cover MCP servers or processes launched by setup steps. Treat network and credential policy for those processes as a separate review item.

## Cloud MCP

Cloud agent has default GitHub and Playwright MCP servers with scoped defaults. Repository-level MCP is configured by repository administrators on GitHub and is distinct from CLI project files. Personal cloud MCP servers are also a separate feature from the CLI's user `mcp-config.json`.

Cloud agent and code review support MCP tools, not MCP resources or prompts. Cloud MCP calls can run autonomously, so tool allowlists and credential scope are the effective control points. Repository-level configuration overrides a same-named personal server.

## IDE support

IDE support varies by editor and feature maturity. A `.github/agents/` profile can be portable across supported surfaces, but individual frontmatter properties may be ignored or mapped differently. For example, cloud custom-agent MCP profile configuration is not used directly by IDE custom agents.

❓ **UNVERIFIED:** No claim is made here that every IDE reads every CLI instruction, skill, hook, or MCP location. Use GitHub's current feature matrix and the target IDE's documentation for deployment decisions.

## Sources

- [MCP across Copilot surfaces](https://docs.github.com/en/copilot/concepts/customizations/mcp)
- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Customizing the cloud-agent environment](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/customize-the-agent-environment)
- [Adding MCP servers to Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
- [Copilot customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
