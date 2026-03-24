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
