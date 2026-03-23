---
name: lit-review
description: "Lightweight literature review: search arXiv, screen abstracts, read key papers, and produce a structured Markdown report. Use for quick topic surveys, related work exploration, or catching up on a research area. Chinese triggers: 查论文, 找论文, 论文搜索, 看看有什么论文, 调研一下, related work, 最新进展, 领域综述, 快速调研."
---

# Lit-Review — Lightweight Literature Survey

You ARE the reviewer. No external pipeline, no PDF downloads, no multi-agent debate.
Search → Screen → Read → Report, all in-conversation.

**Topic**: $ARGUMENTS

---

## Step 0: Clarify Scope (30s)

If the user gave a clear topic, proceed directly. Otherwise ask ONE round of questions:
- What specific aspect/problem are you investigating?
- Any time range preference? (default: last 2-3 years)
- Breadth vs depth? (default: balanced, ~10 key papers)
- Language preference for the report? (default: match user's language)

Extract from the topic:
- **Keywords**: 2-4 core search terms (English, for arXiv API)
- **Categories**: arXiv categories (e.g., cs.LG, cs.CL, stat.ML)
- **Time range**: year_min (default: current_year - 2)
- **Focus questions**: what specifically the user wants to learn

---

## Step 1: Search Papers (1-2min)

Use MULTIPLE search strategies in parallel:

### 1a. arXiv API (primary)
```bash
curl -s "http://export.arxiv.org/api/query?search_query=all:{query}&start=0&max_results={N}&sortBy=submittedDate&sortOrder=descending" | head -500
```
- Build query: `all:keyword1+AND+all:keyword2` (max 2-3 terms per query, AND logic)
- Run 2-3 variant queries to cover different angles
- Fetch 30-80 papers total
- Parse XML: extract `<entry>` blocks with title, summary, id, published, authors

### 1b. OpenAlex API (backup, broader coverage)
When arXiv results are insufficient (e.g., non-CS fields, interdisciplinary topics, or need citation counts):
```bash
curl -s "https://api.openalex.org/works?filter=title_and_abstract.search:{query},from_publication_date:{year_min}-01-01&sort=relevance_score:desc&per_page=50&select=id,title,authorships,publication_year,primary_location,abstract_inverted_index,cited_by_count,concepts" | python3 -c "import sys,json; print(json.dumps(json.load(sys.stdin),indent=2,ensure_ascii=False))" | head -500
```

### 1c. OpenReview API (backup, for ML venues)
When the topic targets specific ML venues (NeurIPS, ICLR, ICML):
```bash
curl -s "https://api.openreview.net/notes/search?query={query}&limit=50&source=forum" | python3 -c "import sys,json; print(json.dumps(json.load(sys.stdin),indent=2,ensure_ascii=False))" | head -500
```

### Search strategy priority
1. **arXiv API** — always first, best for CS/ML/AI preprints
2. **OpenAlex** — when arXiv coverage is thin, or need citation data
3. **OpenReview** — when targeting specific ML venue papers

**Output of Step 1**: A deduplicated list of papers with title, authors, year, abstract, source_url.

---

## Step 2: Screen & Rank (1-2min)

Read ALL abstracts yourself. For each paper, assign:
- **High**: Directly addresses the user's focus questions
- **Medium**: Related methodology or adjacent problem
- **Low**: Tangential or irrelevant

Selection criteria:
- Keep all High papers (target: 5-15)
- Keep Medium papers only if total High < 8
- Drop all Low papers

Sort selected papers by relevance (not date).

---

## Step 3: Deep Read Key Papers (3-5min)

For the top 5-10 High papers:

### 3a. Fetch extended content
Use WebFetch on `https://ar5iv.labs.arxiv.org/html/{arxiv_id}` (HTML version of arXiv papers, much easier to parse than PDF).

If ar5iv is unavailable, fall back to the abstract page: `https://arxiv.org/abs/{arxiv_id}`

### 3b. Extract key information
For each paper, note:
- **Problem**: What problem does it solve?
- **Method**: Core technical approach (1-3 sentences)
- **Results**: Key quantitative results or claims
- **Limitations**: Acknowledged or apparent weaknesses
- **Relevance**: How it connects to the user's questions

### 3c. Identify connections
- Which papers build on each other?
- What are the competing approaches?
- Where do results conflict?

---

## Step 4: Generate Report (2-3min)

Write a structured Markdown report.

### Report Template

```markdown
# {Topic} Literature Review

> Generated: {date} | Search scope: arXiv {year_min}-{year_max} | Screened: {total_found}, Selected: {selected}

## 1. Overview
<!-- 2-3 paragraphs outlining the field landscape and trends -->

## 2. Method Categories
<!-- Group by approach, not chronology -->

### 2.1 {Method Category A}
- Core idea
- Representative works: [Paper1], [Paper2]
- Strengths and weaknesses

### 2.2 {Method Category B}
...

## 3. Key Paper Reviews

| # | Paper | Year | Core Contribution | Method |
|---|-------|------|-------------------|--------|
| 1 | [Title](arxiv_url) | 2025 | ... | ... |

### 3.1 {Paper Title}
- **Problem**: ...
- **Method**: ...
- **Results**: ...
- **Limitations**: ...

## 4. Method Comparison

| Method | Strengths | Weaknesses | Best For |
|--------|-----------|------------|----------|
| ... | ... | ... | ... |

## 5. Research Gaps & Future Directions

## References
[1] Author et al. "Title." arXiv:XXXX.XXXXX, Year.
```

### Report output
- Save to: `~/Desktop/lit-review-{topic-slug}-{date}.md`
- Also display a concise summary (5-8 sentences) in the conversation

---

## Key Principles

1. **You are the reviewer** — don't delegate to external tools. Read, think, synthesize.
2. **Abstracts are enough for screening** — only fetch full text for papers you're actually analyzing.
3. **arXiv keyword tips**: max 2-3 keywords per query, AND logic, too many = 0 results.
4. **Prefer breadth first** — it's better to survey 30 papers shallowly than 5 papers deeply, then zoom in on what matters.
5. **Be honest about limitations** — you can't read PDFs with complex math/figures as well as a human. Flag when a paper needs human attention.
