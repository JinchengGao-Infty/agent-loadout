# MCP Configurations

Pre-configured MCP (Model Context Protocol) server definitions for AI coding agents.

## What is MCP?

MCP allows AI agents to interact with external tools and services through a standardized protocol. Each JSON file here defines one MCP server that your agent can use.

## How to Use

### Claude Code

```bash
# Add an MCP server
claude mcp add <name> -- <command> [args...]

# Or merge into ~/.claude/.mcp.json
```

### Gemini CLI

Add to `.gemini/settings.json` under `mcpServers`.

### Other Agents

Consult your agent's documentation for MCP configuration.

## Available MCPs

| MCP | Description | Setup |
|-----|-------------|-------|
| [ace-tool](ace-tool.json) | Semantic code search across any codebase | Self-host relay ([backend](https://github.com/heromantf/acemcp-relay) + [frontend](https://github.com/heromantf/acemcp-relay-frontend)) + `npm i -g ace-tool` |
| [playwright](playwright.json) | Browser automation and web scraping | `npm install` |
| [drawio](drawio.json) | Create and edit diagrams | `npm install` |
| [memory-palace](memory-palace.json) | Durable cross-session memory | Self-hosted server |
