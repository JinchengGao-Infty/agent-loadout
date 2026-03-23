# How to Write Your Own Skill

Skills are plain Markdown files that teach your AI agent new capabilities. No code required.

## File Structure

```
skills/
  my-skill/
    SKILL.md       # The skill definition (required)
    helper.md      # Supporting docs (optional)
```

## SKILL.md Format

```markdown
---
name: my-skill
description: "One-line description of when to use this skill. Include trigger words for activation."
---

# Skill Title

Instructions for the agent...
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (kebab-case) |
| `description` | Yes | Used by the agent to decide when to activate this skill |

### Writing Good Descriptions

The `description` field is **critical** — it's how your agent decides whether to activate the skill. Tips:

1. **Be specific about triggers**: "Use when the user asks to verify BibTeX entries" not "Academic tool"
2. **Include action verbs**: "Search", "Verify", "Generate", "Debug", "Deploy"
3. **Add multilingual triggers**: If your users speak other languages, add trigger words:
   ```yaml
   description: "... Chinese triggers: 搜索, 查一下, 最新."
   ```
4. **Mention tool names**: If the skill wraps an MCP tool, name it: "Use ACE (mcp__ace-tool__search_context)..."

### Skill Body

The body is free-form Markdown. Common patterns:

- **Step-by-step workflow**: Numbered steps the agent follows
- **Decision tree**: When to do X vs Y
- **Templates**: Output format templates
- **Tool usage**: Specific commands or API calls

### Using Arguments

`$ARGUMENTS` is replaced with whatever the user passes after the skill name:

```markdown
Search for: $ARGUMENTS
```

User types `/my-skill quantum computing` → agent sees `Search for: quantum computing`

## Tips

- Keep skills focused — one skill per task, not a Swiss Army knife
- Include "When to Use" and "When NOT to Use" sections
- Provide concrete examples of commands/API calls
- Use environment variables for secrets: `${API_KEY}` not hardcoded values
- Test your skill by asking your agent to use it

## Platform Compatibility

Skills are just Markdown — they work on any agent that supports file-based skills:

| Platform | Skill Location |
|----------|---------------|
| Claude Code | `~/.claude/skills/` (global) or project-level |
| Gemini CLI | `.gemini/skills/` |
| Codex | `.codex/skills/` |
