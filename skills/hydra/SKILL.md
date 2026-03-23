---
name: hydra
description: "Delegate coding tasks to Codex/Gemini/Claude agents via Hydra — multi-agent git worktree manager with tmux. Use when dispatching coding tasks that need a dedicated agent with isolated workspace. Chinese triggers: 派发任务, 多agent, 并行开发, hydra, 分派, 让codex做, 让gemini做."
---

# Hydra — Multi-Agent Task Manager

Manage coding agents (Codex/Gemini/Claude) via git worktrees and tmux.

> **Prerequisites**: Install Hydra from [github.com/JinchengGao-Infty/hydra](https://github.com/JinchengGao-Infty/hydra) and ensure `hydra` is in your PATH.

## Core Workflow

### 1. Initialize (once per project)
```bash
cd /path/to/project
hydra init
```

### 2. Create Task
```bash
hydra task new "title" --id t001 --allow "**/*"
```

### 3. Open Agent
```bash
# With worktree (parallel tasks)
hydra agent open codex-1 --task t001

# Without worktree (single task)
hydra agent open codex-1 --task t001 --no-worktree
```

### 4. Spawn Agent
```bash
hydra agent spawn codex-1 --task t001 --type codex
```

### 5. Send Prompt
```bash
hydra agent send codex-1 "Read tasks/t001.md and follow all instructions."
```

### 6. Monitor
```bash
hydra agent read codex-1
hydra agent list
```

### 7. Complete
```bash
hydra merge codex-1 --task t001 --squash
hydra agent close codex-1 --task t001 --remove-worktree
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `hydra init` | Initialize project |
| `hydra task new "title" --id ID --allow "globs"` | Create task |
| `hydra task list` | List tasks |
| `hydra agent open NAME --task ID` | Create tmux window |
| `hydra agent spawn NAME --task ID --type codex` | Start agent TUI |
| `hydra agent send NAME "message"` | Send prompt |
| `hydra agent read NAME` | Read output |
| `hydra merge NAME --task ID --squash` | Merge branch |
| `hydra agent close NAME --task ID --remove-worktree` | Cleanup |

## Tips

1. **Don't use `codex --yolo exec`** — that's one-shot, no TUI. Use `hydra agent spawn`.
2. **Don't loop-wait** — check once to confirm startup, then stop.
3. **Use `--no-worktree`** for single non-parallel tasks.
4. **Write task docs to `tasks/`**, not `/tmp`.

## Agent Types
- `codex`: OpenAI Codex
- `gemini`: Gemini CLI
- `claude`: Claude Code
