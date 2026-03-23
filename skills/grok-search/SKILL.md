---
name: grok-search
description: "Search the web for real-time information using dual engines: Grok AI (deep analysis + live web) and Tavily (structured results + web scraping). Use when needing current news, prices, latest releases, or any info beyond training data cutoff. Chinese triggers: 搜索, 搜一下, 查一下, 查查, 看看有没有, 最新, 新闻, grok搜索, 网上搜, 帮我查, 了解一下, 什么价格, 上线了吗, 发布了吗."
---

## Prerequisites

Set these environment variables before use:
- `GROK_API_BASE` — Your Grok-compatible API endpoint (e.g., `http://localhost:8100/v1`)
- `GROK_API_KEY` — Your API key for the Grok endpoint
- `TAVILY_API_KEY` — Your Tavily API key (get one free at [tavily.com](https://tavily.com))

## Instructions

Search for: $ARGUMENTS

---

## Step 1: Do I Need to Search?

**Yes — search when:**
- User explicitly asks for external info
- Real-time data needed (latest versions, recent events, current prices)
- Need to verify knowledge accuracy
- Specific URLs, projects, or products' current status

**No — skip when:**
- Pure code writing/debugging with sufficient context
- User explicitly says not to search

---

## Step 2: Choose Engine

| Scenario | Engine | Why |
|----------|--------|-----|
| Simple fact lookup | Tavily | Fast, structured, scored results |
| Complex / controversial topic | Dual (both) | Cross-verify, reduce hallucination |
| Need AI deep analysis | Grok | Built-in web search + reasoning |
| Latest news | Tavily (topic=news) | News mode optimized |
| Scrape specific URL | Tavily Extract | Full page content as Markdown |
| High-stakes decision | Dual (both) | ≥2 sources required |

---

## Step 3: Execute

### Grok Search (AI-driven, deep analysis)

```bash
curl -s ${GROK_API_BASE}/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${GROK_API_KEY}" \
  -d '{
    "model": "grok-3",
    "messages": [
      {"role": "system", "content": "You are a search assistant. Search the web and provide accurate, well-sourced answers."},
      {"role": "user", "content": "SEARCH_QUERY"}
    ],
    "stream": false
  }' | jq -r '.choices[0].message.content'
```

### Tavily Search (structured results)

```bash
curl -s "https://api.tavily.com/search" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "'${TAVILY_API_KEY}'",
    "query": "SEARCH_QUERY",
    "search_depth": "basic",
    "max_results": 5,
    "include_answer": true
  }' | jq '{answer: .answer, results: [.results[] | {title, url, content, score}]}'
```

**Options:**
- `search_depth`: `basic` (fast) or `advanced` (thorough)
- `topic`: `general` (default), `news`, `finance`
- `max_results`: 1-20
- `time_range`: `day`, `week`, `month`, `year`

### Tavily Extract (scrape a URL)

```bash
curl -s "https://api.tavily.com/extract" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "'${TAVILY_API_KEY}'",
    "urls": ["https://example.com/page"],
    "extract_depth": "basic"
  }' | jq '.results[] | {url, raw_content}'
```

### Dual Search (cross-verify)

Run both in parallel for important queries, then compare results.

---

## Evidence Standards

- **All factual conclusions** need **≥2 independent sources** for cross-verification
- If relying on a single source, **explicitly state** this limitation
- **Prefer:** official docs, Wikipedia, academic databases, authoritative media
- When sources conflict: present both sides, assess credibility and recency
- Confidence levels: **High** (multiple sources agree) · **Medium** (few sources) · **Low** (single source)

## Search Complexity

| Level | Description | Strategy |
|-------|-------------|----------|
| **L1 - Simple** | Single fact, one query | Single engine, direct answer |
| **L2 - Medium** | Multiple aspects | Primary engine + follow-up |
| **L3 - Complex** | Multi-faceted | Dual search + structured planning |
