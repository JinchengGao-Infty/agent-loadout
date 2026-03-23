# Recommended Embedding Models

Some tools in Agent Loadout (like Memory Palace) require a local embedding model for semantic search. Here are our recommendations.

## Top Pick: snowflake-arctic-embed2

Best balance of quality, speed, and resource usage for local deployment.

```bash
# Install via Ollama
ollama pull snowflake-arctic-embed2
```

- **Dimensions**: 1024
- **Context**: 8192 tokens
- **Size**: ~550MB
- **Speed**: Fast on Apple Silicon
- **Quality**: Top-tier for its size class

## Alternatives

| Model | Dims | Size | Best For |
|-------|------|------|----------|
| `snowflake-arctic-embed2` | 1024 | 550MB | General purpose (recommended) |
| `nomic-embed-text` | 768 | 274MB | Lower memory environments |
| `mxbai-embed-large` | 1024 | 670MB | Slightly higher quality |
| `all-minilm` | 384 | 45MB | Minimal resource usage |

## Setup with Ollama

1. Install Ollama: https://ollama.com
2. Pull the model:
   ```bash
   ollama pull snowflake-arctic-embed2
   ```
3. Verify:
   ```bash
   curl http://localhost:11434/api/embeddings -d '{
     "model": "snowflake-arctic-embed2",
     "prompt": "test embedding"
   }'
   ```

## GPU vs CPU

- **Apple Silicon**: Ollama uses Metal GPU acceleration automatically
- **NVIDIA GPU**: Ollama uses CUDA automatically
- **CPU-only**: Works but slower; consider `all-minilm` for speed
