---
name: memory-palace
description: "Manage durable cross-session memory via Memory Palace MCP server. Supports semantic search, write guards, and hierarchical memory organization. Use for saving facts, recalling context, and managing long-term knowledge. Chinese triggers: 记忆, 长期记忆, 记住, 回忆, 召回, 压缩上下文, 重建索引."
---

# Memory Palace

Durable, semantically-searchable memory system for AI agents. Persists knowledge across sessions via a local MCP server.

> **Setup**: See [Memory Palace repository](https://github.com/JinchengGao-Infty/Memory-Palace) for installation. Requires Ollama with an embedding model (recommended: `snowflake-arctic-embed2`).

## First Action in Any Session

Always start with:
```
read_memory("system://boot")
```

This loads core memories and recent context.

## Core Workflow

1. **Boot**: `read_memory("system://boot")`
2. **Search**: `search_memory(query, include_session=true)` if URI is unknown
3. **Read**: `read_memory(uri)` to inspect the exact target
4. **Mutate**: `create_memory` or `update_memory` only after reading
5. **Maintain**: `compact_context` or `rebuild_index` when needed

## Key Rules

- **Read before write**: Always read existing memory before creating/updating
- **Respect guards**: If `guard_action=NOOP`, stop and inspect the suggested target before proceeding
- **Prefer update over create**: Use `update_memory` when a related memory already exists
- **Fresh context**: In new sessions or subagents, always reload with `system://boot`

## Available Tools

| Tool | Purpose |
|------|---------|
| `read_memory` | Read a memory by URI |
| `search_memory` | Semantic search across all memories |
| `create_memory` | Create a new memory |
| `update_memory` | Update an existing memory |
| `delete_memory` | Delete a memory |
| `add_alias` | Add an alternative URI for a memory |
| `compact_context` | Distill noisy session history |
| `rebuild_index` | Rebuild the semantic search index |
| `index_status` | Check index health |

## Common URIs

- `system://boot` — Session initialization
- `system://index` — Full memory index
- `system://recent` — Recently modified memories
- `core://agent` — Agent identity and rules
- `core://my_user` — User profile
