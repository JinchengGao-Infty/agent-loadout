---
name: ace-search
description: "Use ACE (mcp__ace-tool__search_context + enhance_prompt) as the primary code search and prompt enhancement tool. For exploring unfamiliar codebases, finding implementations by behavior, understanding architecture, or before editing code. Prefer over Grep/Glob for semantic and structural queries. Chinese triggers: 代码搜索, 找实现, 找代码, 搜代码, 哪里实现的, 怎么实现的, 代码在哪, 架构, 调用链, 入口在哪, ace搜索."
---

# ACE — Augment Context Engine

ACE provides two tools. Use BOTH, not just search.

## Tool 1: `search_context` — Semantic Code Search

Real-time indexed, cross-language, understands code structure and behavior.

### When to Use (INSTEAD of Grep/Glob)
- Exploring unfamiliar codebases ("where does auth happen?")
- Finding implementations by behavior, not name ("file upload chunk merging")
- Understanding architecture, data flow, or call chains
- **Before editing any code** — find all related symbols first
- **Before starting any task** — recon the codebase to understand scope

### When NOT to Use (use Grep/Glob instead)
- Exact string match (error messages, config values, literal strings)
- Finding ALL references to a known identifier (`grep` guarantees completeness)
- Searching within a specific known file (just `Read` it)

### Query Construction

Format: `Natural language intent + Keywords: specific_identifiers`

**Good queries** — describe behavior, not names:
```
"How the server handles chunk merging during file upload Keywords: upload, chunk, merge, FileService"
"Where cached data is refreshed after user permissions change Keywords: permission, cache, refresh"
"Initialization flow of message queue consumers at startup Keywords: mq, consumer, init, subscribe"
"How configuration hot-reload is triggered and applied Keywords: config, reload, hot update"
```

**Bad queries** — too vague or too literal:
```
"Find class Foo"              → use Grep: class Foo
"Show me foo.py"              → use Read
"Find all references to bar"  → use Grep: bar
```

### Mandatory Workflows

These are not suggestions — treat them as requirements.

#### Pre-Task Reconnaissance
Before starting ANY non-trivial task in an unfamiliar area:
```
search_context(
  project_root_path="/path/to/project",
  query="high-level architecture and main entry points of [feature area] Keywords: [domain terms]"
)
```
This prevents "writing code in the dark" — understand the landscape first.

#### Pre-Edit Deep Dive
Before editing a file, search for ALL symbols involved in the change in ONE call:
```
search_context(
  project_root_path="/path/to/project",
  query="detailed information about [ClassA.methodX], [ClassB], and [InterfaceC] —
         I need to understand how they interact because I'm going to [describe change].
         Keywords: ClassA, methodX, ClassB, InterfaceC"
)
```
One comprehensive query > multiple narrow queries. Include:
- Classes/methods you'll call
- Interfaces you'll implement
- Types you'll use as parameters/return values
- Related tests

#### Bug Investigation
When debugging, search for the behavior, not the symptom:
```
search_context(
  project_root_path="/path/to/project",
  query="how [feature] processes [input type] and what validation/error handling exists Keywords: [error-related terms]"
)
```

---

## Tool 2: `enhance_prompt` — Codebase-Aware Prompt Enhancement

Takes a vague user request, combines it with codebase context and conversation history, and generates a detailed, actionable prompt. Opens a Web UI for user review.

### When to Use
- User message contains `-enhance` or `-enhancer` flag (case-insensitive)
- User explicitly asks to "enhance my prompt" or "improve this requirement"

### Usage
```
enhance_prompt(
  prompt="the user's original request",
  conversation_history="recent 5-10 turns of conversation as string",
  project_root_path="/path/to/project"
)
```

### Workflow
1. User writes a rough requirement (e.g., "Add a login page -enhance")
2. You call `enhance_prompt` with the requirement + conversation context
3. Tool analyzes codebase to understand existing patterns, tech stack, conventions
4. Web UI opens — user reviews and confirms the enhanced prompt
5. You execute based on the enhanced, context-rich prompt

This is especially powerful for:
- New feature requests in unfamiliar codebases
- Ambiguous requirements that need grounding in existing code patterns
- Ensuring new code follows existing conventions

---

## Decision Matrix

| Scenario | Tool |
|----------|------|
| "How is this feature implemented?" | `search_context` |
| "Add a new feature -enhance" | `enhance_prompt` → then `search_context` |
| Find exact string `ERROR_CODE_42` | Grep |
| Find all `*.test.ts` files | Glob |
| Read a known file | Read |
| Understand how module X works | `search_context` |
| About to edit `service.py` | `search_context` first, THEN edit |
| Debug a failing test | `search_context` for the behavior, then Read the test |

---

## Tips

- **Always use absolute paths** for `project_root_path`
- **One rich query > many thin queries** — describe everything you need in one call
- **Describe behavior, not names** — "how auth tokens are validated" beats "find validateToken"
- **Include Keywords for precision** — append concrete identifiers after the natural language description
- **ACE reflects disk state only** — no git history, no uncommitted-but-unstaged changes
