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
1. **Top Pick: Qwen3-Embedding-0.6B** — model specs, install command, why it wins
2. **Comparison table** — Qwen3 vs snowflake-arctic-embed2: dimensions, context length, multilingual, instruction-aware, Chinese retrieval quality, file size
3. **Quantization guide** — fp16 (1.2GB) vs Q8_0 (610MB): negligible quality loss, recommend Q8_0
4. **Reranker: Qwen3-Reranker-0.6B** — what rerankers do (two-stage retrieval), model specs, install command
5. **Alternatives table** — nomic-embed-text (low memory), all-minilm (minimal), Qwen3-Embedding-4B (higher quality)
6. **Setup** — download GGUF files, verify with curl commands (pointing to llama-server, not Ollama)

### File 2: `recommendations/inference-server.md` (new)

Sections:
1. **Recommendation: llama-server (llama.cpp)** — one-line install, native GGUF, Metal/CUDA, dual API endpoints
2. **Comparison table** — llama-server vs Ollama vs TEI vs Infinity: GGUF support, embedding API, reranker API, weight
3. **Quick start: embedding only** — single command to serve Qwen3-Embedding
4. **Quick start: embedding + reranker** — two commands, two ports
5. **Production deployment** — macOS launchd plist example, Linux systemd unit example
6. **Migrating from Ollama** — change port/model in your config, rebuild index

### File 3: `recommendations/embeddings.md` (delete)

Replaced by `embeddings-and-rerankers.md`.

### File 4: `README.md` (edit)

Changes limited to:
- LLM agent instructions block: `ollama pull snowflake-arctic-embed2` → `brew install llama.cpp` + GGUF download commands
- Step 4 Memory Palace: same replacement
- Docs section: update link text and target for embeddings, add inference server link

### File 5: `README_zh.md` (edit)

Mirror all README.md changes in Chinese.

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
