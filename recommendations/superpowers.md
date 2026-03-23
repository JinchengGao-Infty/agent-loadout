# Superpowers Plugin

[Superpowers](https://github.com/obra/superpowers) is a complete software development workflow plugin for AI coding agents. It provides structured brainstorming, TDD, debugging, code review, and implementation planning skills.

## Why Recommend It?

Superpowers gives your agent a disciplined development workflow:
- **Brainstorming** — Explores requirements before jumping into code
- **Test-Driven Development** — Write tests first, then implement
- **Systematic Debugging** — Root cause analysis instead of random fixes
- **Code Review** — Automated quality checks
- **Implementation Planning** — Break complex tasks into manageable steps

## Installation

### Claude Code

```bash
/plugin install superpowers@claude-plugins-official
```

Then enable it in `~/.claude/settings.json`:
```json
{
  "enabledPlugins": {
    "superpowers@claude-plugins-official": true
  }
}
```

### Other Agents

Superpowers skills are pure Markdown files. You can manually copy them from the [Superpowers repo](https://github.com/obra/superpowers) into your agent's skill directory.

## Compatibility

Superpowers works alongside Agent Loadout skills. The skills in this repo (code search, literature review, web search, etc.) complement Superpowers' development workflow skills.
