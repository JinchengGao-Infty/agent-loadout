---
name: grok-imagine
description: "Generate or edit images using a Grok-compatible image API. Use when the user asks to create, draw, edit, or design images, illustrations, logos, or visual content. Chinese triggers: 画, 生成图片, 做图, 画一个, 设计, 图片, 插图, 配图, 头像, logo."
---

## Prerequisites

Set these environment variables:
- `GROK_API_BASE` — Your Grok-compatible API endpoint (e.g., `http://localhost:8100/v1`)
- `GROK_API_KEY` — Your API key

## Instructions

Generate image for: $ARGUMENTS

### Image Generation

```bash
curl -s ${GROK_API_BASE}/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${GROK_API_KEY}" \
  -d '{
    "model": "grok-imagine-1.0",
    "prompt": "IMAGE_DESCRIPTION",
    "n": 1,
    "size": "1024x1024"
  }'
```

The response contains image URLs in `.data[].url`. Download and show to user.

### Image Editing

```bash
curl -s ${GROK_API_BASE}/images/edits \
  -H "Authorization: Bearer ${GROK_API_KEY}" \
  -F "model=grok-imagine-1.0-edit" \
  -F "prompt=EDIT_DESCRIPTION" \
  -F "image=@/path/to/image.png"
```

### Models

- **grok-imagine-1.0** — standard quality generation
- **grok-imagine-1.0-fast** — faster, lower quality
- **grok-imagine-1.0-edit** — edit existing images

### Size Options

Common sizes: `1024x1024`, `1024x1792`, `1792x1024`

### Notes

- If you get "rate_limit_exceeded", the quota is temporarily exhausted. Wait and retry.
- If you get "blocked", the prompt may have triggered content filters. Try rephrasing.
- Generated image URLs are temporary. Download important images to local storage.
