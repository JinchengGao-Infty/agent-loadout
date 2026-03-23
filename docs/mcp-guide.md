# MCP Configuration Guide

MCP (Model Context Protocol) lets AI agents interact with external tools and services through a standardized interface.

## How MCP Works

```
Agent  ←→  MCP Client  ←→  MCP Server  ←→  External Service
                              ↑
                     (defined in config)
```

Your agent's MCP client connects to MCP servers, which provide tools the agent can call. Each server is defined by a command to run or a URL to connect to.

## Configuration by Platform

### Claude Code

**Option A: CLI**
```bash
claude mcp add <name> -- <command> [args...]
claude mcp add <name> --type sse --url <url>
```

**Option B: Config file** (`~/.claude/.mcp.json`)
```json
{
  "mcpServers": {
    "server-name": {
      "command": "executable",
      "args": ["arg1", "arg2"]
    }
  }
}
```

For SSE (Server-Sent Events) servers:
```json
{
  "mcpServers": {
    "server-name": {
      "type": "sse",
      "url": "http://localhost:8765/sse",
      "headers": {
        "X-API-Key": "your-key"
      }
    }
  }
}
```

### Gemini CLI

Add to `.gemini/settings.json`:
```json
{
  "mcpServers": {
    "server-name": {
      "command": "executable",
      "args": ["arg1"]
    }
  }
}
```

### Codex

Add to `.codex/config.json` under `mcpServers`.

## Server Types

### Command-based (stdio)
The agent launches the server as a subprocess. Communication via stdin/stdout.
```json
{
  "command": "npx",
  "args": ["@playwright/mcp@latest"]
}
```

### URL-based (SSE)
The agent connects to an already-running server via HTTP.
```json
{
  "type": "sse",
  "url": "http://localhost:8765/sse"
}
```

## Environment Variables

Some servers need environment variables:
```json
{
  "command": "my-server",
  "env": {
    "API_KEY": "your-key"
  }
}
```

For Claude Code, you can also set env vars globally in `~/.claude/settings.json`:
```json
{
  "env": {
    "GROK_API_BASE": "http://localhost:8100/v1"
  }
}
```

## Troubleshooting

1. **Server not connecting**: Check that the command exists and is in PATH
2. **Permission denied**: Some agents require explicit permission for MCP tools
3. **SSE timeout**: Ensure the server is running before the agent starts
4. **Tool not showing**: Restart the agent after config changes

## Finding More MCPs

- [MCP Registry](https://github.com/modelcontextprotocol) — Official MCP servers
- [Awesome MCP](https://github.com/punkpeye/awesome-mcp-servers) — Community curated list
