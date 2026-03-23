---
name: ace-search
description: "Use ACE (mcp__ace-tool__search_context) as the primary code search tool for exploring unfamiliar codebases, finding implementations by behavior, understanding architecture, or before editing code. Prefer over Grep/Glob for semantic and structural queries. Chinese triggers: 代码搜索, 找实现, 找代码, 搜代码, 哪里实现的, 怎么实现的, 代码在哪, 架构, 调用链, 入口在哪, ace搜索."
---

# ACE Code Search

Use `mcp__ace-tool__search_context` as the **primary** code search tool. It uses semantic indexing and understands code structure across languages.

## When to Use

- Exploring unfamiliar codebases ("where does auth happen?")
- Finding implementations by behavior, not name ("file upload chunk merging")
- Understanding architecture or workflows
- Before editing code — find all related symbols first

## When NOT to Use (use Grep/Glob instead)

- Exact string match (error messages, config values)
- Finding ALL references to a known identifier
- Searching within a specific known file

## Usage

```
search_context(
  project_root_path="/absolute/path/to/project",
  query="natural language description + Keywords: keyword1, keyword2"
)
```

## Tips

- Always use absolute paths for `project_root_path`
- Append `Keywords: ...` to boost precision on specific identifiers
- Combine natural language intent with concrete terms: `"how user authentication works Keywords: login, session, JWT"`
