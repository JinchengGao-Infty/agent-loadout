# Agent Loadout

> Drop-in skills, MCP configs, and workflows for any AI coding agent.

[中文文档](README_zh.md)

**Agent Loadout** is a curated collection of skills and tool configurations that enhance AI coding agents. Clone this repo, point your agent at `SETUP.md`, and it configures itself.

## Quick Start

```bash
git clone https://github.com/JinchengGao-Infty/agent-loadout.git
```

Then tell your agent:

> "Read agent-loadout/SETUP.md and configure yourself with the skills and MCPs."

That's it. Your agent reads the setup guide and installs what it needs.

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| [ace-search](skills/ace-search/) | Semantic code search via ACE — better than grep for understanding codebases |
| [bib-verify](skills/bib-verify/) | Verify BibTeX entries against CrossRef API, detect fabricated references |
| [lit-review](skills/lit-review/) | Lightweight literature review — search arXiv, screen papers, generate reports |
| [grok-search](skills/grok-search/) | Web search with dual engines (Grok AI + Tavily) |
| [grok-imagine](skills/grok-imagine/) | AI image generation and editing via Grok API |
| [hydra](skills/hydra/) | Multi-agent task dispatch via git worktrees and tmux |
| [memory-palace](skills/memory-palace/) | Durable cross-session memory with semantic search |
| [clashverge-rules-debug](skills/clashverge-rules-debug/) | Debug ClashVerge proxy routing on macOS |

### MCP Servers

| MCP | Description |
|-----|-------------|
| [ace-tool](mcp/ace-tool.json) | Semantic code search across any codebase |
| [playwright](mcp/playwright.json) | Browser automation — navigate, click, scrape |
| [drawio](mcp/drawio.json) | Create and edit diagrams |
| [memory-palace](mcp/memory-palace.json) | Persistent agent memory server |

### Recommendations

| Resource | Description |
|----------|-------------|
| [Superpowers](recommendations/superpowers.md) | Development workflow plugin (brainstorming, TDD, debugging) |
| [Embeddings](recommendations/embeddings.md) | Recommended embedding models for semantic search |
| [System Prompt](recommendations/system-prompt.md.example) | Example system prompt for enforcing tool usage |

## Compatibility

Works with any agent that supports file-based skills:

| Platform | Status |
|----------|--------|
| Claude Code | Fully supported |
| Gemini CLI | Fully supported |
| Codex | Fully supported |
| Cursor | Skills via rules, MCP via settings |

## Philosophy

- **Agent-first**: Skills are written for agents to read, not humans
- **Platform-agnostic**: Pure Markdown skills work everywhere
- **Modular**: Pick what you need, skip what you don't
- **No lock-in**: No proprietary formats, no accounts required (except for specific APIs)

## Contributing

1. Fork this repo
2. Add your skill in `skills/your-skill/SKILL.md`
3. Follow the [Skill Authoring Guide](docs/skill-authoring.md)
4. Submit a PR

## License

MIT
