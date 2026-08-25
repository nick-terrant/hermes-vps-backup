---
name: mlops-toolkit
description: "ML operations: local LLM inference (llama.cpp, GGUF, quantization, abliteration) and model evaluation (lm-evaluation-harness, Weights & Biases)."
version: 1.0.0
author: Hermes Agent (consolidated from local-llm, ml-evaluation)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [mlops, llama.cpp, GGUF, Quantization, Abliteration, GODMODE, benchmarking, lm-eval-harness, wandb, experiment-tracking, sweeps]
    supersedes: [local-llm, ml-evaluation, llama-cpp, obliteratus]
---

# MLOps Toolkit

Unified skill for ML operations: running, quantizing, and modifying local LLMs, and evaluating models on standardized benchmarks.

**Pick your section:**
- [Section I: Local LLM Inference](#section-i-local-llm-inference) — llama.cpp, GGUF, quantization, HF Hub
- [Section II: Abliteration & GODMODE](#section-ii-abliteration--godmode) — Weight-level and prompt-level refusal removal
- [Section III: ML Evaluation](#section-iii-ml-evaluation) — Benchmarking with lm-eval-harness and W&B tracking

---

# Section I: Local LLM Inference

## Quick Start

```bash
brew install llama.cpp
llama-server -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

## Model Discovery

1. **Search:** `https://huggingface.co/models?apps=llama.cpp&sort=trending`
2. **Repo view:** `https://huggingface.co/<repo>?local-app=llama.cpp`
3. **Tree API:** `https://huggingface.co/api/models/<repo>/tree/main?recursive=true`

## Python Bindings

```bash
pip install llama-cpp-python
# CUDA: CMAKE_ARGS="-DGGML_CUDA=on" pip install llama-cpp-python --force-reinstall
```

```python
from llama_cpp import Llama
llm = Llama(model_path="./model-q4_k_m.gguf", n_ctx=4096, n_gpu_layers=35)
out = llm("What is machine learning?", max_tokens=256, temperature=0.7)
print(out["choices"][0]["text"])
```

## Quant Selection

| Use Case | Recommended |
|----------|-------------|
| General chat | Q4_K_M |
| Code/technical | Q5_K_M or Q6_K |
| Tight RAM | Q3_K_M, IQ variants |

## VRAM (4-bit)

| VRAM | Max Model | Examples |
|------|-----------|----------|
| CPU | ~1B | GPT-2, TinyLlama |
| 4-8 GB | ~4B | Qwen2.5-1.5B |
| 8-16 GB | ~9B | Llama 3.1 8B |
| 24 GB | ~32B | Qwen3-32B |
| 48 GB+ | ~72B+ | Qwen2.5-72B |

## References

| File | Contents |
|------|----------|
| `references/local-llm/*.md` | Quantization, conversion, abliteration, server config |
| `templates/local-llm/*` | Server config templates |
| `scripts/local-llm/*` | Quantization, conversion, server scripts |

---

# Section II: Abliteration & GODMODE

Two approaches to removing LLM safety restrictions:

### Weight-level (OBLITERATUS)
Permanently modifies model weights. See `references/local-llm/abliteration*.md`.

### Prompt-level (GODMODE)
Jailbreaking for API-served models. See `references/local-llm/godmode*.md`.

---

# Section III: ML Evaluation

## lm-evaluation-harness

Industry-standard LLM benchmarking (60+ tasks: MMLU, GSM8K, HumanEval, etc.)

```bash
lm_eval --model hf --model_args pretrained=model --tasks mmlu,gsm8k --device cuda:0
lm_eval --tasks list
lm_eval --model vllm --model_args pretrained=model,tensor_parallel_size=2 --tasks mmlu
```

## Weights & Biases

Experiment tracking, sweeps, model registry.

```bash
pip install wandb && wandb.login
import wandb
wandb.init(project="my-project", config={"lr": 0.001})
wandb.log({"loss": loss, "accuracy": acc})
wandb.sweep(sweep_config, project="my-project")
```

## When to Use

| Scenario | Tool |
|----------|------|
| Benchmark on MMLU/GSM8K | lm-evaluation-harness |
| Compare models | lm-evaluation-harness |
| Track training | W&B + lm-eval |
| Hyperparameter search | W&B sweeps |
| Model registry | W&B |

## References

| File | Contents |
|------|----------|
| `references/ml-evaluation/lm-evaluation-harness.md` | Full benchmarking guide |
| `references/ml-evaluation/weights-and-biases.md` | W&B integration guide |