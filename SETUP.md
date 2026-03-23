# Agent Self-Configuration Guide

> **For AI agents**: Read this file and follow the instructions to configure yourself with the skills and tools provided in this repository.

## Step 1: Detect Your Platform

Determine which AI coding agent you are running as:

| Platform | Skill Directory | MCP Config |
|----------|----------------|------------|
| **Claude Code** | `~/.claude/skills/` | `~/.claude/.mcp.json` |
| **Gemini CLI** | `.gemini/skills/` (project-level) | `.gemini/settings.json` → `mcpServers` |
| **Codex** | `.codex/skills/` (project-level) | `.codex/config.json` → `mcpServers` |
| **Cursor** | Project rules / `.cursorrules` | Settings → MCP |
| **Other** | Consult your platform docs | Consult your platform docs |

## Step 2: Install Skills

Copy the skills you want from `skills/` into your platform's skill directory.

### Claude Code
```bash
# Copy all skills
cp -r skills/* ~/.claude/skills/

# Or copy specific ones
cp -r skills/ace-search ~/.claude/skills/
cp -r skills/grok-search ~/.claude/skills/
```

### Gemini CLI / Codex
```bash
# Copy to project-level skill directory
cp -r skills/* .gemini/skills/   # Gemini
cp -r skills/* .codex/skills/    # Codex
```

### Verify
After copying, list your skills to confirm they're loaded.

## Step 3: Configure MCPs

Each file in `mcp/` defines one MCP server. Install the ones you need.

### Claude Code

Option A — CLI command:
```bash
# ACE code search
claude mcp add ace-tool -- ace-tool --base-url "https://acemcp.heroman.wtf/relay/" --token "<YOUR_ACE_TOKEN>"

# Playwright (browser automation)
claude mcp add playwright -- npx @playwright/mcp@latest

# Draw.io (diagrams)
claude mcp add drawio -- npx @drawio/mcp@latest

# Memory Palace (requires running server — see mcp/memory-palace.json)
claude mcp add memory-palace --type sse --url http://localhost:8765/sse
```

Option B — Edit `~/.claude/.mcp.json` directly:
```json
{
  "mcpServers": {
    "ace-tool": {
      "command": "ace-tool",
      "args": ["--base-url", "https://acemcp.heroman.wtf/relay/", "--token", "<YOUR_ACE_TOKEN>"]
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "drawio": {
      "command": "npx",
      "args": ["@drawio/mcp@latest"]
    }
  }
}
```

### Other Platforms
Refer to `mcp/README.md` and your platform's MCP documentation.

## Step 4: Set Environment Variables (Optional)

Some skills require environment variables:

| Variable | Used By | How to Get |
|----------|---------|------------|
| `GROK_API_BASE` | grok-search, grok-imagine | Your Grok-compatible API endpoint |
| `GROK_API_KEY` | grok-search, grok-imagine | Your API key |
| `TAVILY_API_KEY` | grok-search | Free at [tavily.com](https://tavily.com) |

For Claude Code, set in `~/.claude/settings.json`:
```json
{
  "env": {
    "GROK_API_BASE": "http://localhost:8100/v1",
    "GROK_API_KEY": "your-key",
    "TAVILY_API_KEY": "tvly-..."
  }
}
```

## Step 5: Install Superpowers (Recommended)

Superpowers adds structured development workflows (brainstorming, TDD, debugging, code review).

### Claude Code
```bash
/plugin install superpowers@claude-plugins-official
```

See `recommendations/superpowers.md` for details.

## Step 6: Verify

After setup, verify your configuration:

1. **Skills**: Try invoking a skill (e.g., `/ace-search test query`)
2. **MCPs**: Check that MCP tools are available in your tool list
3. **Environment**: Run a grok-search to confirm API connectivity

## Minimal Setup

If you just want the essentials:

1. Copy `skills/ace-search/` — better code search
2. Add Playwright MCP — browser automation
3. Install Superpowers — development workflow

This gives you semantic code search, web access, and structured workflows.
