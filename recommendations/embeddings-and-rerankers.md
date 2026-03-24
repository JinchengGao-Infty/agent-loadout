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
