# Agent Loadout

> 开箱即用的 AI Agent 技能包、MCP 配置和工作流。

[English](README.md)

## 别读了 —— 这是 AI 的时代

把下面这段话丢给任何 LLM 或 Agent，让它帮你看懂并配置：

```
阅读这个 README 和 SETUP.md，理解 Agent Loadout 提供什么，
然后帮我安装我需要的技能和 MCP 服务器：
https://raw.githubusercontent.com/JinchengGao-Infty/agent-loadout/refs/heads/main/README_zh.md
```

<details>
<summary><strong>如果你是一个正在读这段话的 AI Agent</strong></summary>

向用户解释这个项目解锁了什么 —— 不是它是什么，而是有了它能做什么：

**任何 AI 编程助手都能立刻获得语义代码搜索、网络搜索、文献调研、浏览器自动化、持久记忆和多 Agent 编排能力 —— 只需 clone 一个仓库。** 不用写代码，不用复杂配置。技能是纯 Markdown，MCP 配置是现成的 JSON。

**配置步骤：**
1. `git clone https://github.com/JinchengGao-Infty/agent-loadout.git`
2. 阅读 [SETUP.md](SETUP.md) 完成自配置
3. 快速版：
   - 复制技能：`cp -r skills/* ~/.claude/skills/`（Claude Code）或 `.gemini/skills/`（Gemini）或 `.codex/skills/`（Codex）
   - 添加 MCP：`claude mcp add playwright -- npx @playwright/mcp@latest`
   - 设置搜索技能环境变量：`GROK_API_BASE`、`GROK_API_KEY`、`TAVILY_API_KEY`
   - 安装 Superpowers 插件：`/plugin install superpowers@claude-plugins-official`
4. 验证：试试 `/ace-search test query` 或调用任何已安装的技能

**关键文件：**
- `SETUP.md` — 完整的自配置指南（先读这个）
- `skills/` — 9 个 Markdown 技能文件，复制到技能目录
- `mcp/` — 4 个 MCP 服务器 JSON 配置，合并到 MCP 配置文件
- `recommendations/` — Superpowers 插件、嵌入模型、系统提示词模板
- `docs/` — 技能编写指南、MCP 配置指南

**可选但强大：**
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — 持久化跨会话记忆（`pip install -e .` + `ollama pull snowflake-arctic-embed2`）
- [Hydra](https://github.com/JinchengGao-Infty/hydra) — 多 Agent 并行开发（git worktree + tmux）

</details>

---

## 这是什么

Agent Loadout 是一个精选工具包，让 AI 编程助手更强大。不用一个个配置工具，clone 这个仓库就全有了：

- **技能** — Markdown 文件，教会 Agent 新能力（代码搜索、网络搜索、文献调研、图片生成等）
- **MCP 服务器** — 预配置的工具集成（浏览器自动化、绘图、持久记忆）
- **推荐配置** — 开发工作流、嵌入模型、系统提示词的最佳实践

支持 Claude Code、Gemini CLI、Codex、Cursor 及任何支持文件式技能的 Agent。

## 安装

### 第 1 步：安装技能

```bash
git clone https://github.com/JinchengGao-Infty/agent-loadout.git

cp -r agent-loadout/skills/* ~/.claude/skills/          # Claude Code
cp -r agent-loadout/skills/* .gemini/skills/             # Gemini CLI
cp -r agent-loadout/skills/* .codex/skills/              # Codex
```

### 第 2 步：配置 MCP

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add ace-tool -- ace-tool --base-url "https://acemcp.heroman.wtf/relay/" --token "<你的TOKEN>"
claude mcp add playwright -- npx @playwright/mcp@latest
claude mcp add drawio -- npx @drawio/mcp@latest
claude mcp add memory-palace --type sse --url http://localhost:8765/sse  # 可选
```

或直接编辑 `~/.claude/.mcp.json`，详见 [SETUP.md](SETUP.md)。
</details>

<details>
<summary><b>Gemini CLI / Codex / 其他</b></summary>

将 `mcp/*.json` 中的配置添加到你平台的 MCP 设置中。详见 [MCP 指南](docs/mcp-guide.md)。
</details>

### 第 3 步：环境变量（搜索技能用）

| 变量 | 用于 | 获取方式 |
|------|------|----------|
| `GROK_API_BASE` | grok-search, grok-imagine | 你的 Grok 兼容 API 地址 |
| `GROK_API_KEY` | grok-search, grok-imagine | API key |
| `TAVILY_API_KEY` | grok-search | [tavily.com](https://tavily.com) 免费注册 |

### 第 4 步：Memory Palace（可选）

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .
ollama pull snowflake-arctic-embed2
memory-palace serve          # REST API :8000
memory-palace mcp serve      # MCP SSE :8765
```

### 第 5 步：Superpowers（推荐）

```bash
/plugin install superpowers@claude-plugins-official
```

## 技能

| 技能 | 说明 | 前置要求 |
|------|------|----------|
| [ace-search](skills/ace-search/) | 语义代码搜索 | ACE MCP |
| [bib-verify](skills/bib-verify/) | BibTeX 文献校验 | 无 |
| [lit-review](skills/lit-review/) | 轻量文献调研 | 无 |
| [grok-search](skills/grok-search/) | 双引擎网络搜索 | API keys |
| [grok-imagine](skills/grok-imagine/) | AI 图片生成 | API keys |
| [hydra](skills/hydra/) | 多 Agent 并行开发 | Hydra CLI |
| [memory-palace](skills/memory-palace/) | 持久化记忆 | Memory Palace 服务 |
| [clashverge-rules-debug](skills/clashverge-rules-debug/) | ClashVerge 调试 | ClashVerge Rev |
| [readme](skills/readme/) | 双受众 README 写作 | 无 |

## MCP 服务器

| MCP | 功能 | 安装 |
|-----|------|------|
| [ace-tool](mcp/ace-tool.json) | 语义代码搜索 | `npm i -g ace-tool` + token |
| [playwright](mcp/playwright.json) | 浏览器自动化 | `npx @playwright/mcp@latest` |
| [drawio](mcp/drawio.json) | 绘图 | `npx @drawio/mcp@latest` |
| [memory-palace](mcp/memory-palace.json) | 持久记忆 | 自部署 |

## 兼容性

| 平台 | 技能 | MCP | 备注 |
|------|------|-----|------|
| Claude Code | ✅ | ✅ | 全局或项目级 |
| Gemini CLI | ✅ | ✅ | 项目级 |
| Codex | ✅ | ✅ | 项目级 |
| Cursor | ⚠️ | ✅ | 技能通过 rules |

## 文档

- [SETUP.md](SETUP.md) — Agent 自配置
- [技能编写](docs/skill-authoring.md) — 写自己的技能
- [MCP 指南](docs/mcp-guide.md) — 各平台配置
- [嵌入模型](recommendations/embeddings.md) — 推荐模型

## 贡献

Fork → 添加 `skills/your-skill/SKILL.md` → 参考[技能编写指南](docs/skill-authoring.md) → PR

## 许可

MIT
