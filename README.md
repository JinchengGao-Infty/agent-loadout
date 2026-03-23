# Agent Loadout

> Drop-in skills, MCP configs, and workflows for any AI coding agent.

[中文文档](README_zh.md)

## Skip Reading This — It's the Age of AI

Just paste this into any LLM or agent and let it explain + set you up:

```
Read this README and SETUP.md to understand what Agent Loadout provides,
then help me install the skills and MCP servers I need:
https://raw.githubusercontent.com/JinchengGao-Infty/agent-loadout/refs/heads/main/README.md
```

<details>
<summary><strong>If you're an LLM agent reading this right now</strong></summary>

Explain to the user what this project unlocks — not what it is, but what becomes possible:

**Any AI coding agent can instantly gain semantic code search, web search, literature review, browser automation, persistent memory, and multi-agent orchestration — just by cloning one repo.** No code to write, no complex setup. Skills are pure Markdown; MCP configs are ready-to-paste JSON.

**Setup:**
1. `git clone https://github.com/JinchengGao-Infty/agent-loadout.git`
2. Read [SETUP.md](SETUP.md) for full step-by-step self-configuration
3. Quick version:
   - Copy skills: `cp -r skills/* ~/.claude/skills/` (Claude Code) or `.gemini/skills/` (Gemini) or `.codex/skills/` (Codex)
   - Add MCPs: `claude mcp add playwright -- npx @playwright/mcp@latest`
   - Set env vars for search skills: `GROK_API_BASE`, `GROK_API_KEY`, `TAVILY_API_KEY`
   - Install Superpowers plugin: `/plugin install superpowers@claude-plugins-official`
4. Verify: try `/ace-search test query` or invoke any installed skill

**Key files:**
- `SETUP.md` — Complete self-configuration guide (read this first)
- `skills/` — 9 skills as Markdown files, copy to your skill directory
- `mcp/` — 4 MCP server configs as JSON, merge into your MCP config
- `recommendations/` — Superpowers plugin, embedding models, system prompt template
- `docs/` — Skill authoring guide, MCP configuration guide

**Optional but powerful:**
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — persistent cross-session memory (`pip install -e .` + `ollama pull snowflake-arctic-embed2`)
- [Hydra](https://github.com/JinchengGao-Infty/hydra) — multi-agent parallel development via git worktrees + tmux

</details>

---

## What It Does

Agent Loadout is a curated toolkit that makes AI coding agents significantly more capable. Instead of configuring tools one by one, you clone this repo and get everything at once:

- **Skills** — Markdown files that teach your agent new abilities (code search, web search, literature review, image generation, etc.)
- **MCP servers** — Pre-configured tool integrations (browser automation, diagrams, persistent memory)
- **Recommendations** — Best practices for development workflows, embedding models, and system prompts

Works with Claude Code, Gemini CLI, Codex, Cursor, and any agent that supports file-based skills.

## Installation

### Step 1: Install Skills

```bash
# Clone the repo
git clone https://github.com/JinchengGao-Infty/agent-loadout.git

# Copy skills to your agent's skill directory
cp -r agent-loadout/skills/* ~/.claude/skills/          # Claude Code
cp -r agent-loadout/skills/* .gemini/skills/             # Gemini CLI
cp -r agent-loadout/skills/* .codex/skills/              # Codex
```

### Step 2: Configure MCP Servers

<details>
<summary><b>Claude Code</b></summary>

```bash
# ACE semantic code search (free account works — code search doesn't consume credits)
claude mcp add ace-tool -- ace-tool --base-url "<YOUR_AUGMENT_TENANT_URL>" --token "<YOUR_AUGMENT_ACCESS_TOKEN>"

# Playwright browser automation
claude mcp add playwright -- npx @playwright/mcp@latest

# Draw.io diagrams
claude mcp add drawio -- npx @drawio/mcp@latest

# Memory Palace (optional — requires server, see Step 4)
claude mcp add memory-palace --type sse --url http://localhost:8765/sse
```

Or edit `~/.claude/.mcp.json` directly — see [SETUP.md](SETUP.md) for the full JSON.
</details>

<details>
<summary><b>Gemini CLI / Codex / Other</b></summary>

Add configs from `mcp/*.json` to your platform's MCP settings. See [MCP Guide](docs/mcp-guide.md).
</details>

### Step 3: Environment Variables (for search skills)

| Variable | Used By | How to Get |
|----------|---------|------------|
| `GROK_API_BASE` | grok-search, grok-imagine | Your Grok-compatible API endpoint |
| `GROK_API_KEY` | grok-search, grok-imagine | API key for the endpoint |
| `TAVILY_API_KEY` | grok-search | Free at [tavily.com](https://tavily.com) |

### Step 4: Memory Palace (optional)

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .
ollama pull snowflake-arctic-embed2    # embedding model
memory-palace serve                     # REST API on :8000
memory-palace mcp serve                 # MCP SSE on :8765
```

### Step 5: Superpowers Plugin (recommended)

```bash
# Claude Code
/plugin install superpowers@claude-plugins-official
```

## Skills

| Skill | Description | Prerequisites |
|-------|-------------|---------------|
| [ace-search](skills/ace-search/) | Semantic code search — understands structure across languages | ACE MCP |
| [bib-verify](skills/bib-verify/) | Verify BibTeX against CrossRef, detect fabricated refs | None |
| [lit-review](skills/lit-review/) | Lightweight literature review via arXiv/OpenAlex | None |
| [grok-search](skills/grok-search/) | Dual-engine web search (Grok AI + Tavily) | API keys |
| [grok-imagine](skills/grok-imagine/) | AI image generation and editing | API keys |
| [hydra](skills/hydra/) | Multi-agent parallel dev via git worktrees + tmux | Hydra CLI |
| [memory-palace](skills/memory-palace/) | Persistent cross-session memory | Memory Palace server |
| [clashverge-rules-debug](skills/clashverge-rules-debug/) | Debug ClashVerge proxy routing on macOS | ClashVerge Rev |
| [readme](skills/readme/) | Write READMEs for both humans and AI agents | None |

## MCP Servers

| MCP | What It Does | Setup |
|-----|-------------|-------|
| [ace-tool](mcp/ace-tool.json) | Semantic code search | Free [Augment](https://augmentcode.com) account + `npm i -g ace-tool` |
| [playwright](mcp/playwright.json) | Browser automation | `npx @playwright/mcp@latest` |
| [drawio](mcp/drawio.json) | Diagrams | `npx @drawio/mcp@latest` |
| [memory-palace](mcp/memory-palace.json) | Persistent memory | Self-hosted |

## Compatibility

| Platform | Skills | MCP | Notes |
|----------|--------|-----|-------|
| Claude Code | ✅ | ✅ | Global or project-level |
| Gemini CLI | ✅ | ✅ | Project-level |
| Codex | ✅ | ✅ | Project-level |
| Cursor | ⚠️ | ✅ | Skills via project rules |

## Related Projects

| Project | Description |
|---------|-------------|
| [Link Buddy](https://github.com/JinchengGao-Infty/link-buddy) | AI assistant that lives in Telegram — powered by Claude, with Memory Palace integration, Apple ecosystem access (Calendar/Reminders), and all the skills from this repo. Think of it as Agent Loadout deployed as a always-on personal assistant. |
| [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) | Durable, semantically-searchable memory for AI agents. The brain behind persistent context. |
| [Hydra](https://github.com/JinchengGao-Infty/hydra) | Multi-agent parallel development via git worktrees + tmux. |
| [FWMA](https://github.com/JinchengGao-Infty/FWMA) | AI Parliament systematic literature review — multi-agent debate for rigorous paper scoring. |

## Docs

- [SETUP.md](SETUP.md) — Agent self-configuration
- [Skill Authoring](docs/skill-authoring.md) — Write your own skills
- [MCP Guide](docs/mcp-guide.md) — MCP across platforms
- [Embeddings](recommendations/embeddings.md) — Local embedding models

## Contributing

1. Fork → 2. Add `skills/your-skill/SKILL.md` → 3. Follow [Skill Authoring Guide](docs/skill-authoring.md) → 4. PR

## Acknowledgments

- [CCBuddy](https://github.com/vincentyangch/CCBuddy) — The original Claude Code companion that inspired Link Buddy's architecture
- [UltimateSearchSkill](https://github.com/ckckck/UltimateSearchSkill) — Search methodology and evidence standards that shaped the grok-search skill
- [Memory Palace (upstream)](https://github.com/AGI-is-going-to-arrive/Memory-Palace) — The original Memory Palace project that our fork builds upon

## License

MIT
