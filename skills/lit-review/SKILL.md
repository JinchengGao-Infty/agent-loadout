---
name: lit-review
description: "Lightweight literature review: search arXiv/Semantic Scholar/OpenAlex/PubMed, screen abstracts, read key papers, and produce a structured Markdown report. Use for quick topic surveys, related work exploration, or catching up on a research area. Chinese triggers: 查论文, 找论文, 论文搜索, 看看有什么论文, 调研一下, related work, 最新进展, 领域综述, 快速调研."
---

# Lit-Review — Lightweight Literature Survey

You ARE the reviewer. No external pipeline, no PDF downloads, no multi-agent debate.
Search → Screen → Read → Report, all in-conversation.

**Topic**: $ARGUMENTS

---

## Step 0: Clarify Scope & Prepare Keywords (30s)

If the user gave a clear topic, proceed directly. Otherwise ask ONE round of questions:
- What specific aspect/problem are you investigating?
- Any time range preference? (default: last 2-3 years)
- Breadth vs depth? (default: balanced, ~10 key papers)
- Language preference for the report? (default: match user's language)

Extract from the topic:
- **Keywords**: 2-4 core search terms (English, for API queries)
- **Categories**: arXiv categories (e.g., cs.LG, cs.CL, stat.ML)
- **Time range**: year_min (default: current_year - 2)
- **Focus questions**: what specifically the user wants to learn

### Keyword Translation & Expansion

If the user provides non-English keywords:
1. **Translate** to academic English phrases (2-8 words each). Preserve domain abbreviations (CNN, SVM, RAG, etc.).
2. **Expand** each keyword with synonyms, abbreviations, hypernyms/hyponyms. E.g.:
   - "检索增强生成" → `retrieval augmented generation` → expand: `RAG`, `retrieval-augmented LLM`, `grounded generation`
   - "思维链" → `chain of thought` → expand: `CoT`, `step-by-step reasoning`, `reasoning chain`
3. Use expanded terms to build variant queries for broader coverage.

---

## Step 1: Search Papers (1-2min)

Use MULTIPLE search strategies in parallel. Run at least 2-3 sources for any topic.

### 1a. arXiv API (primary for CS/ML/AI/Physics/Math)
```bash
curl -s "http://export.arxiv.org/api/query?search_query=all:{query}&start=0&max_results={N}&sortBy=submittedDate&sortOrder=descending" | head -500
```
- Build query: `all:keyword1+AND+all:keyword2` (max 2-3 terms per query, AND logic)
- Run 2-3 variant queries using expanded keywords to cover different angles
- Fetch 30-80 papers total
- Parse XML: extract `<entry>` blocks with title, summary, id, published, authors

### 1b. Semantic Scholar Bulk API (primary for citation data + broad coverage)
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search/bulk?query={query}&limit=100&fields=title,year,citationCount,abstract,venue,externalIds,openAccessPdf&year={year_min}-{year_max}&sort=citationCount:desc"
```
- **Free, no API key needed**. Use `/search/bulk` endpoint (NOT `/search` which requires key).
- Returns: title, abstract, year, venue, citationCount, DOI, ArXiv ID, PDF URL
- `sort=citationCount:desc` to get highly-cited papers first
- **Note**: `tldr` field is NOT available on bulk endpoint; use abstract instead
- Pagination: response includes `token` for next batch if needed
- **ExternalIds**: `externalIds.DOI`, `externalIds.ArXiv` for cross-referencing

### 1c. OpenAlex API (broad coverage, good for non-CS fields)
```bash
curl -s "https://api.openalex.org/works?filter=title_and_abstract.search:{query},from_publication_date:{year_min}-01-01&sort=relevance_score:desc&per_page=50&select=id,title,authorships,publication_year,primary_location,abstract_inverted_index,cited_by_count,concepts"
```
- Free, no API key needed (polite pool: add `mailto=user@example.com` for faster rate)
- Covers journals, conferences, preprints
- `abstract_inverted_index` reconstruction: `" ".join(w for w,_ in sorted(((w,p) for w,pos in idx.items() for p in pos), key=lambda x:x[1]))`

### 1d. PubMed E-utilities (for biomedical/clinical topics)
When the topic touches medicine, biology, clinical, health, or neuroscience:
```bash
# Step 1: Search for IDs
curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term={query}&retmax=30&retmode=json&sort=relevance"
# Step 2: Fetch details (batch up to 20 IDs comma-separated)
curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id={id1},{id2},...&retmode=xml"
```
- Free, 10 req/s without API key
- Parse XML: `ArticleTitle`, `Abstract/AbstractText`, `Journal/Title`, `ArticleId[@IdType="doi"]`
- Essential for medical/bio topics where arXiv has zero coverage

### 1e. OpenReview API (for ML venues with review scores)
When targeting NeurIPS, ICLR, ICML, EMNLP:
```bash
curl -s "https://api.openreview.net/notes/search?query={query}&limit=50&source=forum"
```
- Filter by venue: `&venue=ICLR.cc/2025/Conference`

### Search strategy priority
1. **arXiv + Semantic Scholar** — always first, run in parallel
2. **OpenAlex** — when arXiv coverage is thin, or for journal papers
3. **PubMed** — when topic involves biomedical/clinical
4. **OpenReview** — when targeting specific ML venue papers

---

## Step 1.5: Merge, Deduplicate & Enrich

After collecting results from all sources, merge into a unified list:

### Deduplication (by priority)
1. **DOI match**: Normalize DOIs to lowercase, exact match → merge
2. **ArXiv ID match**: Same `arxiv_id` → merge
3. **Title match**: Lowercase + strip punctuation, fuzzy match (>90% similarity) → merge
4. When merging duplicates, **keep the richest fields**: prefer S2 abstract > OpenAlex abstract, prefer S2 citationCount, keep all source URLs

### Crossref Enrichment (for papers missing DOI)
For papers without DOI (typically from arXiv):
```bash
curl -s "https://api.crossref.org/works?query.bibliographic={title}&rows=1&select=DOI,title,author,container-title,abstract"
```
- Match by title similarity before accepting the DOI
- Fills in: DOI, journal name, sometimes abstract
- Free, polite pool (add `mailto` header for better rate)

### Output
A unified, deduplicated table:
| Field | Source priority |
|-------|----------------|
| title | any (first found) |
| abstract | S2 > PubMed > OpenAlex > arXiv |
| citationCount | S2 > OpenAlex |
| DOI | S2 > Crossref > OpenAlex |
| arxiv_id | arXiv > S2 |
| pdf_url | S2 openAccessPdf > arXiv |
| venue | S2 > OpenAlex > PubMed |

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
- **Citation boost**: Papers with >100 citations in the same relevance tier get priority

Sort selected papers by relevance (not date).

---

## Step 3: Deep Read Key Papers (3-5min)

For the top 5-10 High papers:

### 3a. Fetch extended content
Use WebFetch on `https://ar5iv.labs.arxiv.org/html/{arxiv_id}` (HTML version of arXiv papers, much easier to parse than PDF).

If ar5iv is unavailable, fall back to the abstract page: `https://arxiv.org/abs/{arxiv_id}`

For non-arXiv papers: use the DOI URL or publisher page via WebFetch.

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

> Generated: {date} | Sources: {sources_used} | Found: {total_found} → Deduped: {after_dedup} → Selected: {selected}

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

| # | Paper | Year | Citations | Core Contribution | Method |
|---|-------|------|-----------|-------------------|--------|
| 1 | [Title](url) | 2025 | 123 | ... | ... |

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
[1] Author et al. "Title." Venue, Year. DOI: xxx
```

### Report output
- Save to: `~/Desktop/lit-review-{topic-slug}-{date}.md`
- Also display a concise summary (5-8 sentences) in the conversation

---

## Key Principles

1. **You are the reviewer** — don't delegate to external tools. Read, think, synthesize.
2. **Abstracts are enough for screening** — only fetch full text for papers you're actually analyzing.
3. **arXiv keyword tips**: max 2-3 keywords per query, AND logic, too many = 0 results.
4. **Always use 2+ sources** — single-source reviews miss papers. arXiv + S2 is the minimum.
5. **Keyword expansion matters** — translate and expand before searching. One extra synonym can surface 20 more relevant papers.
6. **Deduplicate before screening** — don't waste time reading the same paper from different sources.
7. **Prefer breadth first** — survey 30 papers shallowly, then zoom in on what matters.
8. **Be honest about limitations** — flag when a paper needs human attention.
