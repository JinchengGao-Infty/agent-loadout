# Agent Loadout

> 开箱即用的 AI Agent 技能包、MCP 配置和工作流。

[English](README.md)

**Agent Loadout** 是一个精选的 AI 编程助手增强工具集。Clone 仓库，让你的 Agent 读 `SETUP.md`，它会自己完成配置。

## 快速开始

```bash
git clone https://github.com/JinchengGao-Infty/agent-loadout.git
```

然后告诉你的 Agent：

> "读 agent-loadout/SETUP.md，按照里面的说明配置你自己。"

就这样。Agent 会自己读指南、安装需要的技能和工具。

## 包含内容

### 技能 (Skills)

| 技能 | 说明 |
|------|------|
| [ace-search](skills/ace-search/) | 语义代码搜索 — 比 grep 更懂代码结构 |
| [bib-verify](skills/bib-verify/) | BibTeX 文献校验，检测伪造引用 |
| [lit-review](skills/lit-review/) | 轻量文献调研 — 搜索 arXiv、筛选论文、生成报告 |
| [grok-search](skills/grok-search/) | 双引擎网络搜索（Grok AI + Tavily） |
| [grok-imagine](skills/grok-imagine/) | AI 图片生成和编辑 |
| [hydra](skills/hydra/) | 多 Agent 并行任务管理（git worktree + tmux） |
| [memory-palace](skills/memory-palace/) | 持久化跨会话记忆，支持语义搜索 |
| [clashverge-rules-debug](skills/clashverge-rules-debug/) | macOS ClashVerge 代理分流调试 |

### MCP 服务器

| MCP | 说明 |
|-----|------|
| [ace-tool](mcp/ace-tool.json) | 语义代码搜索 |
| [playwright](mcp/playwright.json) | 浏览器自动化 |
| [drawio](mcp/drawio.json) | 绘图工具 |
| [memory-palace](mcp/memory-palace.json) | Agent 持久记忆服务 |

### 推荐配置

| 资源 | 说明 |
|------|------|
| [Superpowers](recommendations/superpowers.md) | 开发工作流插件（头脑风暴、TDD、调试） |
| [Embeddings](recommendations/embeddings.md) | 推荐的本地嵌入模型 |
| [System Prompt](recommendations/system-prompt.md.example) | 系统提示词模板 |

## 兼容性

适用于所有支持文件式技能的 Agent：

| 平台 | 状态 |
|------|------|
| Claude Code | 完全支持 |
| Gemini CLI | 完全支持 |
| Codex | 完全支持 |
| Cursor | 技能通过 rules，MCP 通过设置 |

## 设计理念

- **Agent 优先**：技能是写给 Agent 读的，不是给人读的
- **平台无关**：纯 Markdown 技能，到处能用
- **模块化**：按需选用，不强制全装
- **无锁定**：没有专有格式，不需要注册账号（特定 API 除外）

## 贡献

1. Fork 本仓库
2. 在 `skills/your-skill/SKILL.md` 添加技能
3. 参考 [技能编写指南](docs/skill-authoring.md)
4. 提交 PR

## 许可

MIT
