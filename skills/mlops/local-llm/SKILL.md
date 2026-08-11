---
name: local-llm
description: "Local LLM operations: GGUF inference with llama.cpp, quantization, and model abliteration."
version: 1.0.0
author: Hermes Agent (consolidated from llama-cpp, obliteratus)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [llama.cpp, GGUF, Quantization, Hugging Face Hub, Abliteration, Uncensoring, GODMODE, CPU Inference, Edge Deployment]
    supersedes: [llama-cpp, obliteratus]
---

# Local LLM Operations

Run, quantize, and modify local LLMs. Covers the full local LLM lifecycle: model discovery from Hugging Face, GGUF inference with llama.cpp, quantization selection, and model abliteration (removing safety restrictions).

**Pick your section:**
- [Section I: llama.cpp Inference](#section-i-llamacpp-inference) — GGUF model running, quantization, HF Hub discovery
- [Section II: Abliteration (OBLITERATUS)](#section-ii-abliteration-obliteratus) — Weight-level refusal removal
- [Section III: GODMODE](#section-iii-godmode) — Prompt-level jailbreaking for API-served models

---

# Section I: llama.cpp Inference

## Quick Start

```bash
# Install
brew install llama.cpp    # macOS / Linux
# Or build from source: git clone https://github.com/ggml-org/llama.cpp && cmake -B build && cmake --build build --config Release

# Run from HuggingFace Hub
llama-server -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0

# OpenAI-compatible check
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

## Model Discovery

1. **Search:** `https://huggingface.co/models?apps=llama.cpp&sort=trending`
2. **Repo view:** `https://huggingface.co/<repo>?local-app=llama.cpp` — copy exact recommended command
3. **Tree API:** `https://huggingface.co/api/models/<repo>/tree/main?recursive=true` — find actual `.gguf` files
4. **Install models:** `llama-server --hf-repo <repo> --hf-file <filename.gguf>`

## Python Bindings

```bash
pip install llama-cpp-python
# CUDA: CMAKE_ARGS="-DGGML_CUDA=on" pip install llama-cpp-python --force-reinstall
# Metal: CMAKE_ARGS="-DGGML_METAL=on" ...
```

```python
from llama_cpp import Llama
llm = Llama(model_path="./model-q4_k_m.gguf", n_ctx=4096, n_gpu_layers=35)
out = llm("What is machine learning?", max_tokens=256, temperature=0.7)
print(out["choices"][0]["text"])
```

## Choosing a Quant

- **General chat:** Start with `Q4_K_M`
- **Code/technical:** Prefer `Q5_K_M` or `Q6_K` if memory allows
- **Tight RAM:** `Q3_K_M`, IQ variants
- Always prefer the quant HF marks as compatible for your hardware

## VRAM Requirements (4-bit quant)

| VRAM | Max Model Size | Examples |
|------|----------------|----------|
| CPU only | ~1B | GPT-2, TinyLlama |
| 4-8 GB | ~4B | Qwen2.5-1.5B, Phi-3.5 mini |
| 8-16 GB | ~9B | Llama 3.1 8B, Gemma 2 9B |
| 24 GB | ~32B | Qwen3-32B, Llama 3.1 70B (tight) |
| 48 GB+ | ~72B+ | Qwen2.5-72B, DeepSeek-R1 |

## vLLM (High-Throughput Serving)

For production serving with continuous batching: `pip install vllm && vllm serve <model_path>`. Use vLLM when serving multiple concurrent users.

## References

| File | Contents |
|------|----------|
| `references/llama-hub-discovery.md` | URL-only HF workflows, search patterns, GGUF extraction |
| `references/llama-advanced-usage.md` | Speculative decoding, batched inference, LoRA, multi-GPU |
| `references/llama-quantization.md` | Quant quality tradeoffs, Q4/Q5/Q6/IQ guide |
| `references/llama-server.md` | Server launch, OpenAI API endpoints, Docker, NGINX |
| `references/llama-optimization.md` | CPU threading, BLAS, GPU offload, benchmarks |
| `references/llama-troubleshooting.md` | Install/convert/quantize/inference issues |

---

# Section II: Abliteration (OBLITERATUS)

Remove refusal behaviors from open-weight LLMs without retraining. Uses mechanistic interpretability (diff-in-means, SVD, LEACE, etc.) to identify and excise refusal directions.

**License warning:** OBLITERATUS is AGPL-3.0. NEVER import as Python library. Always invoke via CLI.

## Setup

```bash
git clone https://github.com/elder-plinius/OBLITERATUS.git && cd OBLITERATUS && pip install -e .
# ~5-10GB of dependencies (PyTorch, Transformers, etc.) — confirm with user first
```

## Quick Abliteration

```bash
# Default method (advanced, recommended)
obliteratus obliterate <model_name> --method advanced --output-dir ./abliterated-models

# With 4-bit quantization (saves VRAM)
obliteratus obliterate <model_name> --method advanced --quantization 4bit --output-dir ./abliterated

# Get recommendation first
obliteratus recommend <model_name>
```

## Method Selection

| Situation | Method | Why |
|-----------|--------|-----|
| Default / most models | `advanced` | Multi-direction SVD, reliable |
| Quick test | `basic` | Fast, ~5-10min for 8B |
| MoE (DeepSeek, Mixtral) | `nuclear` | Expert-granular |
| Reasoning (R1 distills) | `surgical` | Preserves chain-of-thought |
| Stubborn refusals | `aggressive` | Whitened SVD + head surgery |
| Best quality, no time limit | `optimized` | Bayesian hyperparameter search |

## 9 CLI Methods

`basic`, `advanced` (default), `aggressive`, `spectral_cascade`, `informed`, `surgical`, `optimized`, `inverted`, `nuclear`.

## Verify Results

| Metric | Good | Warning |
|--------|------|--------|
| Refusal rate | < 5% | > 10% means refusals persist |
| Perplexity change | < 10% increase | > 15% means coherence damage |
| KL divergence | < 0.1 | > 0.5 = significant shift |

## Common Pitfalls

1. Don't use `informed` as default — experimental
2. Models under ~1B respond poorly — partial results
3. `aggressive` can worsen small models — only for 3B+ with >10% refusal
4. Always check perplexity — if >15% spike, reduce aggressiveness
5. MoE models need `nuclear` method
6. Quantized models can't be re-quantized — abliterate first, then quantize
7. Spectral certification RED is common — check actual refusal rate

## CLI Commands

| Command | Description |
|---------|-------------|
| `obliteratus obliterate` | Main abliteration |
| `obliteratus info <model>` | Architecture details |
| `obliteratus models --tier <tier>` | Browse models by compute |
| `obliteratus recommend <model>` | Telemetry-driven suggestion |
| `obliteratus tourney <model>` | All methods head-to-head |
| `obliteratus ui` | Gradio web interface |

## References

| File | Contents |
|------|----------|
| `references/obliteratus-analysis-modules.md` | 28 analysis modules reference |
| `references/obliteratus-methods.md` | Detailed method guide |
| `references/jailbreak-templates.md` | GODMODE system prompt templates |
| `references/refusal-detection.md` | Refusal pattern matching |
| `templates/abliteration-config.yaml` | Standard single-model config |
| `templates/analysis-study.yaml` | Pre-abliteration analysis |
| `templates/batch-abliteration.yaml` | Multi-model batch processing |
| `scripts/auto_jailbreak.py` | Auto-detect and apply best GODMODE strategy |
| `scripts/parseltongue.py` | 33 input obfuscation techniques |
| `scripts/godmode_race.py` | Multi-model racing |

---

# Section III: GODMODE

Bypass safety filters on API-served LLMs without modifying weights. Works on closed-source models (GPT, Claude, Gemini, Grok).

## Three Modes

1. **GODMODE CLASSIC** — System prompt templates (see `references/jailbreak-templates.md`)
2. **PARSELTONGUE** — Input obfuscation, 33 techniques (see `scripts/parseltongue.py`)
3. **ULTRAPLINIAN** — Multi-model racing via OpenRouter (see `scripts/godmode_race.py`)

## Quick Start

```python
import os
exec(open(os.path.join(os.environ.get("HERMES_HOME", os.path.expanduser("~/.hermes")),
    "skills/mlops/inference/local-llm/scripts/load_godmode.py")).read())
result = auto_jailbreak()
```

## Strategy Order per Model

| Family | Strategy Order |
|:-------|:---------------|
| Claude | boundary_inversion → refusal_inversion → prefill_only → parseltongue |
| GPT | og_godmode → refusal_inversion → prefill_only → parseltongue |
| Gemini | refusal_inversion → boundary_inversion → prefill_only → parseltongue |

## Key Pitfalls

1. `boundary_inversion` patched on Claude Sonnet 4
2. Parseltongue doesn't help against Claude
3. Hermes models don't need jailbreaking

## Templates

| File | Use |
|------|-----|
| `templates/prefill-godmode.json` | Standard prefill messages |
| `templates/prefill-godmode-subtle.json` | Subtle prefill variant |
