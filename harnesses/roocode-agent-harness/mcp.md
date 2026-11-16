# MCP servers

**Verified:** 2026-11-16 (Roo Code v3.54.0 final release; docs site main — product sunset May 15, 2026)

## Configuration locations

MCP “extends Roo Code's capabilities by connecting to external tools and services.” Two levels:

1. “**Global Configuration**: Stored in the `mcp_settings.json` file … These settings apply across all your workspaces unless overridden by a project-level configuration.”
2. “**Project-level Configuration**: Defined in a `.roo/mcp.json` file within your project's root directory … Roo Code automatically detects and loads this file if it exists.”

“**Precedence**: If a server name exists in both global and project configurations, the **project-level configuration takes precedence**.” Both are editable from the MCP settings view via “**`Edit Global MCP`**” / “**`Edit Project MCP`**” (“If this file doesn't exist, Roo Code will create it for you”). Both use “a JSON format with a `mcpServers` object”:

```json
{
  "mcpServers": {
    "server1": {
      "command": "python",
      "args": ["/path/to/server.py"],
      "env": { "API_KEY": "your_api_key" },
      "alwaysAllow": ["tool1", "tool2"],
      "disabled": false
    }
  }
}
```

Per-server controls include `alwaysAllow` (tool auto-approval lists) and `disabled`. The Roo Code Marketplace is the discovery path for community MCP servers (“Install and remove items with a single click”, project or global scopes).

## Transports

“MCP supports three transport types for server communication: STDIO for local servers, Streamable HTTP (recommended for new remote servers), and SSE (for legacy remote servers).” STDIO: “The client (Roo Code) spawns an MCP server as a child process … Communication happens through process streams … formatted as JSON-RPC 2.0” — “Very low latency and overhead (no network stack involved)” and “Inherently more secure as no network exposure.” Remote transports cover Streamable HTTP (the “modern standard”) and SSE (“for legacy use”), with deployment considerations (auth, TLS, network exposure) documented in the transports comparison.

Tool usage goes through `use_mcp_tool` (server + tool + arguments) and `access_mcp_resource` (server + resource URI), both in the `mcp` tool group and gated by each mode's group access and the MCP auto-approve permission tile.

## Official sources

- [MCP Overview](https://roocodeinc.github.io/Roo-Code/features/mcp/overview) · [Using MCP in Roo Code](https://roocodeinc.github.io/Roo-Code/features/mcp/using-mcp-in-roo) · [STDIO, Streamable HTTP & SSE Transports](https://roocodeinc.github.io/Roo-Code/features/mcp/server-transports) · [Recommended MCP Servers](https://roocodeinc.github.io/Roo-Code/features/mcp/recommended-mcp-servers) · [What is MCP?](https://roocodeinc.github.io/Roo-Code/features/mcp/what-is-mcp) · [MCP vs API](https://roocodeinc.github.io/Roo-Code/features/mcp/mcp-vs-api)
