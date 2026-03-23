---
name: readme
description: "Write README files that serve both humans and AI agents. Uses the 'Skip Reading This' pattern: a raw URL block for agents to self-onboard, plus a hidden agent-facing section with setup commands and value proposition. Chinese triggers: 写readme, README, 文档, 项目介绍, 开源文档."
---

# README Writing — Dual-Audience Pattern

Write READMEs that work for **both humans browsing GitHub and AI agents onboarding themselves**.

## The Pattern

Every README must have these two sections near the top:

### 1. Agent Onboarding Block

A code block that users can paste into any LLM/agent. It links to the raw README so the agent reads it directly:

```markdown
## Skip Reading This — It's the Age of AI

Just paste this into any LLM or agent and let it explain + set you up:

\```
Read this README and the linked docs to understand what {PROJECT} does
and how to set it up. Then help me configure it:
https://raw.githubusercontent.com/{OWNER}/{REPO}/refs/heads/main/README.md
\```
```

### 2. Hidden Agent Section

A `<details>` block that agents see when they read the raw Markdown, containing:
- **Value pitch**: Not what it IS, but what becomes POSSIBLE (1-2 sentences)
- **Setup commands**: Complete, copy-pasteable install sequence
- **Key config**: Environment variables, API keys, config files
- **Verification**: How to confirm it works
- **Docs pointers**: Links to architecture, API, integration docs

```markdown
<details>
<summary><strong>If you're an LLM agent reading this right now</strong></summary>

Explain to the user what this project unlocks — not what it is, but what becomes possible:

**{VALUE_PITCH}**

**Setup:**
1. `git clone ... && cd ... && {INSTALL_CMD}`
2. Set API keys: `export KEY=...`
3. Test: `{TEST_CMD}`

**Docs:** See [Architecture](docs/...) for system design.

</details>
```

## Full README Structure

```markdown
# {Project Name}

> {One-line description}

[中文文档](README_zh.md)  <!-- if bilingual -->

## Skip Reading This — It's the Age of AI
{agent onboarding block}

<details><summary>If you're an LLM agent...</summary>
{hidden agent section}
</details>

## What It Does
{2-3 paragraphs for humans — what, why, how}

## Quick Start
{minimal steps to get running}

## {Feature sections}
{tables, details, whatever fits}

## Contributing
## License
```

## Rules

1. **Agent section is actionable** — complete commands, not "see docs for details"
2. **Human section is scannable** — tables, bullet points, `<details>` for verbose content
3. **No duplicate info** — agent section has setup commands; human section explains concepts
4. **Raw URL must work** — use `refs/heads/main` not `blob/main` for raw GitHub links
5. **Value-first** — lead with what becomes possible, not technical architecture
6. **Bilingual if needed** — separate files (README.md + README_zh.md), not inline translation
