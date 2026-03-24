# Embedding & Reranker Infrastructure Update

**Date:** 2026-03-24
**Status:** Draft

## Goal

Update Agent Loadout's embedding/reranker recommendations and inference server guidance. Replace snowflake-arctic-embed2 + Ollama with Qwen3 GGUF models + llama-server. Add reranker support (previously absent).

## Motivation

1. **Qwen3-Embedding-0.6B outperforms snowflake-arctic-embed2** in Chinese retrieval (3x better discrimination) and multilingual coverage (100+ languages vs English-focused). Same dimensionality (1024), smaller file (610MB Q8_0 vs 1.2GB).
2. **Rerankers dramatically improve retrieval quality** — relevant docs score 0.99+, irrelevant 0.0001. Agent Loadout had no reranker guidance at all.
3. **llama-server supports both `/v1/embeddings` and `/v1/rerank`** with native GGUF. Ollama has no rerank API. llama-server is equally easy to install (`brew install llama.cpp`).
4. **GGUF is the emerging standard** for local model deployment — universal format, quantization-friendly, supported by all major runtimes.

## Changes

### File 1: `recommendations/embeddings-and-rerankers.md` (new, replaces `embeddings.md`)

Sections:
1. **Top Pick: Qwen3-Embedding-0.6B** — model specs (Q8_0, 610MB, 1024 dim, 32K context), why it wins
2. **Comparison table** — Qwen3 vs snowflake-arctic-embed2: dimensions, context length, multilingual, instruction-aware, Chinese retrieval quality, file size
3. **Quantization guide** — fp16 (1.2GB) vs Q8_0 (610MB): negligible quality loss, recommend Q8_0
4. **Reranker: Qwen3-Reranker-0.6B** — what rerankers do (two-stage: embedding recall → reranker precision), model specs (Q8_0, 610MB), total stack = 1.2GB for both models
5. **Alternatives table** — nomic-embed-text (low memory), all-minilm (minimal), Qwen3-Embedding-4B (higher quality)
6. **Setup** — GGUF download from HuggingFace (`Qwen/Qwen3-Embedding-0.6B-GGUF`, `ggml-org/Qwen3-Reranker-0.6B-Q8_0-GGUF`), curl verification against llama-server endpoints (port 11435 for embedding, 11436 for reranker)

### File 2: `recommendations/inference-server.md` (new)

Sections:
1. **Recommendation: llama-server (llama.cpp)** — `brew install llama.cpp` (macOS) / build from source (Linux), native GGUF, Metal/CUDA, OpenAI-compatible endpoints
2. **Comparison table** — llama-server vs Ollama vs TEI vs Infinity: columns = GGUF support, `/v1/embeddings`, `/v1/rerank`, install weight (binary size / deps)
3. **Quick start: embedding only** — `llama-server -m model.gguf --embedding --pooling last --port 11435 -ngl 99`
4. **Quick start: embedding + reranker** — two processes, ports 11435 + 11436
5. **Production deployment** — macOS launchd plist (KeepAlive + RunAtLoad, log paths), Linux systemd unit (Restart=always). Health check: `curl localhost:PORT/health`
6. **Migrating from Ollama** — update API base URL (11434→11435), change model name, rebuild embedding index. Note: if using Memory Palace, existing embeddings are incompatible across models — must rebuild via `rebuild_index` MCP tool or API

### File 3: `recommendations/embeddings.md` (delete)

Replaced by `embeddings-and-rerankers.md`.

### File 4: `README.md` (edit)

Three targeted changes:
- **LLM agent `<details>` block** (line ~42): replace `ollama pull snowflake-arctic-embed2` with `brew install llama.cpp` + GGUF download curl commands. Remove old Ollama instructions entirely (Ollama covered as fallback in inference-server.md).
- **Step 4 Memory Palace** (line ~109-117): same replacement — llama-server commands instead of ollama
- **Docs section** (line ~172): `[Embeddings](recommendations/embeddings.md)` → `[Embeddings & Rerankers](recommendations/embeddings-and-rerankers.md)`, add `[Inference Server](recommendations/inference-server.md)`

### File 5: `README_zh.md` (edit)

Mirror all README.md changes. Straight translation, no expanded Chinese-specific content (the models/commands are the same).

## Out of Scope

- No changes to `skills/`, `mcp/`, `docs/skill-authoring.md`, `docs/mcp-guide.md`
- No changes to `SETUP.md` (it doesn't reference specific embedding models)
- No new skills or MCP configs
- No benchmark raw data (conclusions only)

## Implementation Order

1. Write `recommendations/embeddings-and-rerankers.md`
2. Write `recommendations/inference-server.md`
3. Delete `recommendations/embeddings.md`
4. Update `README.md`
5. Update `README_zh.md`
6. Commit and push
