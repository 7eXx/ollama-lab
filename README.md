# ollama-lab

Local LLM with Ollama in Docker, used for coding through opencode.

Machine: HP ZBook G1a, Ryzen 7 AI 350 CPU (Radeon 860M iGPU, gfx1150).

## Structure

```
ollama-lab/
├── docker-compose.yml
└── README.md
```

## Getting started

```bash
docker compose up -d
docker exec -it ollama ollama pull qwen2.5-coder:14b
docker exec -it ollama ollama ps
```

The `PROCESSOR` column of `ollama ps` should show the GPU, not the CPU.

## Configuration

- Ollama API: `http://127.0.0.1:11434` (exposed locally only).
- Models are stored in the `ollama` Docker volume (mounted at `/root/.ollama` inside the container).
- Main environment variables in the compose file:
  - `HSA_OVERRIDE_GFX_VERSION=11.0.0`: forces the architecture for ROCm, since gfx1150 is not officially supported.
  - `OLLAMA_CONTEXT_LENGTH=32768`: large context window, required by opencode.
  - `OLLAMA_FLASH_ATTENTION=1` and `OLLAMA_KV_CACHE_TYPE=q8_0`: lower memory usage for the cache.

## Recommended coding models

| Model | Notes |
|---|---|
| `qwen3-coder:30b` | MoE, great with tools, needs about 20 GB of RAM |
| `qwen2.5-coder:14b` | lighter, less reliable tool calling |
| `qwen2.5-coder:7b` | fast, good for quick tests and simple tasks |

## opencode

Configuration in `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Local Ollama",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": {
        "qwen2.5-coder:14b": { "name": "Qwen2.5 Coder 14B" },
        "qwen3-coder:30b": { "name": "Qwen3 Coder 30B" }
      }
    }
  }
}
```

Then, in the folder of the project you want to work on, run `opencode` and pick the model with `/models`.

## Troubleshooting

- **Runs on CPU instead of GPU**: check `docker logs ollama | grep -i -E "gpu|rocm|vulkan"`. Try `HSA_OVERRIDE_GFX_VERSION=11.0.2`, or use the plain `ollama/ollama` image with `OLLAMA_VULKAN=1` and only `/dev/dri`.
- **opencode answers poorly or hangs**: check the context size (`OLLAMA_CONTEXT_LENGTH`) and try a model with better tool calling.
- **Model too large**: pick a smaller model or increase the memory reserved for the iGPU in the BIOS.