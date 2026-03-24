# Embedding & Reranker Infrastructure Update — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Update Agent Loadout's embedding/reranker recommendations from Ollama + snowflake-arctic-embed2 to llama-server + Qwen3 GGUF, and add reranker guidance.

**Architecture:** Documentation-only changes across 4 files (2 new, 2 edited) plus 1 deletion. No code, no tests.

**Tech Stack:** Markdown

**Spec:** `docs/superpowers/specs/2026-03-24-embedding-reranker-infra-update-design.md`

---

### Task 1: Write `recommendations/embeddings-and-rerankers.md`

**Files:**
- Create: `recommendations/embeddings-and-rerankers.md`

- [ ] **Step 1: Write the file**

```markdown
# Recommended Embedding & Reranker Models

Some tools in Agent Loadout (like Memory Palace) use local embedding and reranker models for semantic search. Here are our recommendations.

## Top Pick: Qwen3-Embedding-0.6B

Best balance of quality, multilingual coverage, and resource usage.

| Spec | Value |
|------|-------|
| **Format** | GGUF (Q8_0) |
| **Size** | 610 MB |
| **Dimensions** | 1024 |
| **Context** | 32,768 tokens |
| **Languages** | 100+ (including code) |
| **Instruction-aware** | Yes |

```bash
# Download from HuggingFace
curl -L -o qwen3-embedding-0.6b-q8_0.gguf \
  "https://huggingface.co/Qwen/Qwen3-Embedding-0.6B-GGUF/resolve/main/Qwen3-Embedding-0.6B-Q8_0.gguf"
```

### Why Qwen3 over snowflake-arctic-embed2?

| | Qwen3-Embedding-0.6B | snowflake-arctic-embed2 |
|---|---|---|
| **Dimensions** | 1024 | 1024 |
| **Context** | 32K | 8K |
| **Size (Q8_0)** | 610 MB | 1.2 GB |
| **Languages** | 100+ | English-focused |
| **Instruction-aware** | Yes | No |
| **Chinese retrieval** | Strong (MTEB-zh 66.3) | Weak |
| **MRL (flexible dims)** | Yes (32-1024) | No |

Qwen3 is smaller, supports 4x longer context, and significantly better at Chinese and multilingual retrieval. Q8_0 quantization has negligible quality loss vs fp16.

### Quantization

| Variant | Size | Quality vs fp16 | Recommendation |
|---------|------|-----------------|----------------|
| Q8_0 | 610 MB | ~99.5% | **Recommended** — best size/quality tradeoff |
| fp16 | 1.2 GB | 100% (baseline) | Only if you need maximum precision |

## Reranker: Qwen3-Reranker-0.6B

Rerankers are the second stage of a two-stage retrieval pipeline:

1. **Embedding** (fast, approximate): encode query + all docs → cosine similarity → top-N candidates
2. **Reranker** (precise, slower): score each candidate against the query → re-order by relevance

Adding a reranker dramatically improves precision — relevant documents score 0.99+, irrelevant ones drop to near zero.

| Spec | Value |
|------|-------|
| **Format** | GGUF (Q8_0) |
| **Size** | 610 MB |
| **Context** | 32,768 tokens |
| **Languages** | 100+ |

```bash
# Download from HuggingFace
curl -L -o qwen3-reranker-0.6b-q8_0.gguf \
  "https://huggingface.co/ggml-org/Qwen3-Reranker-0.6B-Q8_0-GGUF/resolve/main/qwen3-reranker-0.6b-q8_0.gguf"
```

**Total stack: 1.2 GB** for both embedding + reranker (same as snowflake-arctic-embed2 alone, but with reranker included).

## Alternatives

| Model | Dims | Size | Best For |
|-------|------|------|----------|
| `Qwen3-Embedding-0.6B` | 1024 | 610 MB | **General purpose (recommended)** |
| `Qwen3-Embedding-4B` | 1024 | 2.5 GB | Higher quality, more resources |
| `nomic-embed-text` | 768 | 274 MB | Lower memory environments |
| `all-minilm` | 384 | 45 MB | Minimal resource usage |

## Serving

See [Inference Server](inference-server.md) for how to serve these models with llama-server.

### Quick Verification

After starting llama-server (see inference server guide), verify with:

```bash
# Test embedding
curl -s http://127.0.0.1:11435/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3-embedding","input":"test embedding"}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'OK: {len(d[\"data\"][0][\"embedding\"])} dims')"

# Test reranker
curl -s http://127.0.0.1:11436/v1/rerank \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3-reranker","query":"what is memory palace","documents":["Memory Palace is a persistent memory system for AI agents","The weather is nice today"]}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print(f'doc {r[\"index\"]}: {r[\"relevance_score\"]:.4f}') for r in d['results']]"
```
```

- [ ] **Step 2: Verify markdown renders correctly**

Run: `cat recommendations/embeddings-and-rerankers.md | head -5`
Expected: file exists and starts with `# Recommended Embedding & Reranker Models`

- [ ] **Step 3: Commit**

```bash
git add recommendations/embeddings-and-rerankers.md
git commit -m "docs: add Qwen3 embedding + reranker recommendations (replaces snowflake)"
```

---

### Task 2: Write `recommendations/inference-server.md`

**Files:**
- Create: `recommendations/inference-server.md`

- [ ] **Step 1: Write the file**

```markdown
# Inference Server for Embedding & Reranker

How to serve GGUF embedding and reranker models locally.

## Recommended: llama-server (llama.cpp)

llama-server is the only lightweight server that supports **both** `/v1/embeddings` and `/v1/rerank` with native GGUF format.

```bash
# macOS
brew install llama.cpp

# Linux — build from source
git clone https://github.com/ggml-org/llama.cpp && cd llama.cpp
cmake -B build && cmake --build build --config Release
# Binary at build/bin/llama-server
```

## Why llama-server?

| | llama-server | Ollama | TEI | Infinity |
|---|---|---|---|---|
| **GGUF support** | ✅ native | ✅ | ❌ safetensors | ❌ safetensors |
| **`/v1/embeddings`** | ✅ | ✅ | ✅ | ✅ |
| **`/v1/rerank`** | ✅ | ❌ | ✅ | ✅ |
| **Install** | `brew install` / single binary | App / CLI | brew / docker | pip (Python) |
| **GPU** | Metal + CUDA | Metal + CUDA | Metal + CUDA | MPS + CUDA |
| **Weight** | ~137 MB binary | ~100 MB + runtime | ~26 MB binary | ~500 MB + PyTorch |

Ollama is simpler but **has no reranker API**. TEI and Infinity don't support GGUF. llama-server is the sweet spot.

## Quick Start: Embedding Only

```bash
llama-server \
  -m qwen3-embedding-0.6b-q8_0.gguf \
  --embedding --pooling last \
  --port 11435 --host 127.0.0.1 \
  -ngl 99
```

- `--embedding` — enable embedding mode
- `--pooling last` — use last-token pooling (required for Qwen3)
- `-ngl 99` — offload all layers to GPU

## Quick Start: Embedding + Reranker

Run two processes on different ports:

```bash
# Terminal 1: Embedding server
llama-server \
  -m qwen3-embedding-0.6b-q8_0.gguf \
  --embedding --pooling last \
  --port 11435 --host 127.0.0.1 \
  -ngl 99

# Terminal 2: Reranker server
llama-server \
  -m qwen3-reranker-0.6b-q8_0.gguf \
  --reranking \
  --port 11436 --host 127.0.0.1 \
  -ngl 99
```

Health check: `curl http://127.0.0.1:11435/health` → `{"status":"ok"}`

## Production Deployment

### macOS (launchd)

Create `~/Library/LaunchAgents/com.llama-embedding.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.llama-embedding</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/llama-server</string>
        <string>-m</string>
        <string>/path/to/qwen3-embedding-0.6b-q8_0.gguf</string>
        <string>--embedding</string>
        <string>--pooling</string>
        <string>last</string>
        <string>--port</string>
        <string>11435</string>
        <string>--host</string>
        <string>127.0.0.1</string>
        <string>-ngl</string>
        <string>99</string>
        <string>--log-disable</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/llama-embedding.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/llama-embedding.err.log</string>
</dict>
</plist>
```

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.llama-embedding.plist
```

Create a similar plist for the reranker on port 11436 with `--reranking` instead of `--embedding --pooling last`.

### Linux (systemd)

Create `/etc/systemd/system/llama-embedding.service`:

```ini
[Unit]
Description=llama-server embedding (Qwen3-Embedding-0.6B)
After=network.target

[Service]
ExecStart=/usr/local/bin/llama-server \
  -m /path/to/qwen3-embedding-0.6b-q8_0.gguf \
  --embedding --pooling last \
  --port 11435 --host 127.0.0.1 \
  -ngl 99 --log-disable
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now llama-embedding
```

## Migrating from Ollama

If you were using Ollama for embeddings:

1. **Install llama-server**: `brew install llama.cpp`
2. **Download GGUF models**: see [Embeddings & Rerankers](embeddings-and-rerankers.md)
3. **Update your config**: change the embedding API base URL from `http://127.0.0.1:11434/v1` (Ollama) to `http://127.0.0.1:11435/v1` (llama-server), and update the model name
4. **Rebuild index**: if using Memory Palace, existing embeddings are from a different model and must be rebuilt — use the `rebuild_index` MCP tool or call `POST /rebuild-index` on the REST API
5. **Optional**: stop Ollama if no longer needed

> **Note:** Ollama remains a valid choice if you only need embeddings (no reranker). It's simpler to set up but limited — see the comparison table above.
```

- [ ] **Step 2: Verify**

Run: `cat recommendations/inference-server.md | head -5`
Expected: file exists and starts with `# Inference Server`

- [ ] **Step 3: Commit**

```bash
git add recommendations/inference-server.md
git commit -m "docs: add inference server guide (llama-server vs Ollama/TEI/Infinity)"
```

---

### Task 3: Delete `recommendations/embeddings.md`

**Files:**
- Delete: `recommendations/embeddings.md`

- [ ] **Step 1: Delete the old file**

```bash
git rm recommendations/embeddings.md
```

- [ ] **Step 2: Commit**

```bash
git commit -m "docs: remove old embeddings.md (replaced by embeddings-and-rerankers.md)"
```

---

### Task 4: Update `README.md`

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update LLM agent details block (line ~42)**

Replace:
```markdown
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — persistent cross-session memory (`pip install -e .` + `ollama pull snowflake-arctic-embed2`)
```
With:
```markdown
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — persistent cross-session memory (`pip install -e .` + `brew install llama.cpp` + [download GGUF models](recommendations/embeddings-and-rerankers.md))
```

- [ ] **Step 2: Update Step 4 Memory Palace (lines ~109-117)**

Replace:
```markdown
### Step 4: Memory Palace (optional)

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .
ollama pull snowflake-arctic-embed2    # embedding model
memory-palace serve                     # REST API on :8000
memory-palace mcp serve                 # MCP SSE on :8765
```
```
With:
```markdown
### Step 4: Memory Palace (optional)

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .

# Install inference server + download embedding model
brew install llama.cpp
curl -L -o qwen3-embedding.gguf \
  "https://huggingface.co/Qwen/Qwen3-Embedding-0.6B-GGUF/resolve/main/Qwen3-Embedding-0.6B-Q8_0.gguf"
llama-server -m qwen3-embedding.gguf --embedding --pooling last --port 11435 -ngl 99 &

memory-palace serve                     # REST API on :8000
memory-palace mcp serve                 # MCP SSE on :8765
```

See [Embeddings & Rerankers](recommendations/embeddings-and-rerankers.md) for reranker setup and model alternatives.
See [Inference Server](recommendations/inference-server.md) for production deployment (launchd/systemd).
```

- [ ] **Step 3: Update Docs section (line ~172)**

Replace:
```markdown
- [Embeddings](recommendations/embeddings.md) — Local embedding models
```
With:
```markdown
- [Embeddings & Rerankers](recommendations/embeddings-and-rerankers.md) — Recommended models
- [Inference Server](recommendations/inference-server.md) — Serving models locally
```

- [ ] **Step 4: Verify links**

Run: `grep -n 'embeddings.md\|embeddings-and-rerankers.md\|inference-server.md' README.md`
Expected: no references to old `embeddings.md`, new links present

- [ ] **Step 5: Commit**

```bash
git add README.md
git commit -m "docs: update README — llama-server + Qwen3 replaces Ollama + snowflake"
```

---

### Task 5: Update `README_zh.md`

**Files:**
- Modify: `README_zh.md`

- [ ] **Step 1: Update LLM agent details block (line ~42)**

Replace:
```markdown
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — 持久化跨会话记忆（`pip install -e .` + `ollama pull snowflake-arctic-embed2`）
```
With:
```markdown
- [Memory Palace](https://github.com/JinchengGao-Infty/Memory-Palace) — 持久化跨会话记忆（`pip install -e .` + `brew install llama.cpp` + [下载 GGUF 模型](recommendations/embeddings-and-rerankers.md)）
```

- [ ] **Step 2: Update Step 4 Memory Palace (lines ~101-109)**

Replace:
```markdown
### 第 4 步：Memory Palace（可选）

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .
ollama pull snowflake-arctic-embed2
memory-palace serve          # REST API :8000
memory-palace mcp serve      # MCP SSE :8765
```
```
With:
```markdown
### 第 4 步：Memory Palace（可选）

```bash
git clone https://github.com/JinchengGao-Infty/Memory-Palace.git
cd Memory-Palace && pip install -e .

# 安装推理服务器 + 下载嵌入模型
brew install llama.cpp
curl -L -o qwen3-embedding.gguf \
  "https://huggingface.co/Qwen/Qwen3-Embedding-0.6B-GGUF/resolve/main/Qwen3-Embedding-0.6B-Q8_0.gguf"
llama-server -m qwen3-embedding.gguf --embedding --pooling last --port 11435 -ngl 99 &

memory-palace serve          # REST API :8000
memory-palace mcp serve      # MCP SSE :8765
```

详见 [嵌入与重排序模型](recommendations/embeddings-and-rerankers.md) 了解 reranker 配置和备选模型。
详见 [推理服务器](recommendations/inference-server.md) 了解生产部署（launchd/systemd）。
```

- [ ] **Step 3: Update Docs section (line ~163)**

Replace:
```markdown
- [嵌入模型](recommendations/embeddings.md) — 推荐模型
```
With:
```markdown
- [嵌入与重排序模型](recommendations/embeddings-and-rerankers.md) — 推荐模型
- [推理服务器](recommendations/inference-server.md) — 本地模型服务
```

- [ ] **Step 4: Verify**

Run: `grep -n 'embeddings.md\|embeddings-and-rerankers.md\|inference-server.md' README_zh.md`
Expected: no references to old `embeddings.md`, new links present

- [ ] **Step 5: Commit**

```bash
git add README_zh.md
git commit -m "docs: update README_zh — 同步 llama-server + Qwen3 更新"
```

---

### Task 6: Final push

- [ ] **Step 1: Review all changes**

```bash
git log --oneline -6
```
Expected: 5 new commits (tasks 1-5)

- [ ] **Step 2: Push**

```bash
git push origin main
```
