---
name: ml-evaluation
description: "Unified ML evaluation toolkit: lm-evaluation-harness for benchmarking LLMs on 60+ academic tasks (MMLU, GSM8K, HumanEval, etc.), and Weights & Biases for experiment tracking, hyperparameter sweeps, and model registry management."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [mlops, evaluation, benchmarking, lm-eval-harness, wandb, experiment-tracking, sweeps, model-registry]
    category: mlops
---

# ML Evaluation

A consolidated skill covering two complementary evaluation domains: standardized LLM benchmarking with lm-evaluation-harness, and full-lifecycle experiment tracking with Weights & Biases.

## lm-evaluation-harness

Industry-standard benchmarking suite (EleutherAI, HuggingFace, major labs) for evaluating LLMs across 60+ academic benchmarks.

**Capabilities:**
- Evaluate on MMLU, GSM8K, HumanEval, TruthfulQA, HellaSwag, ARC, and 50+ more tasks
- Supports HuggingFace, vLLM, and API-based model backends
- Compare multiple models with standardized metrics
- Track training progress across checkpoints
- vLLM backend for 5-10× faster inference

**Quick commands:**
| Action | Command |
|--------|---------|
| Basic evaluation | `lm_eval --model hf --model_args pretrained=model --tasks mmlu,gsm8k --device cuda:0` |
| List tasks | `lm_eval --tasks list` |
| vLLM (fast) | `lm_eval --model vllm --model_args pretrained=model,tensor_parallel_size=2 --tasks mmlu` |
| Compare models | Evaluate multiple models, generate comparison table |

**Common workflows:**
1. **Standard benchmark evaluation** — choose suite → configure model → run → analyze results
2. **Training progress tracking** — periodic eval on quick benchmarks (HellaSwag ~10min)
3. **Model comparison** — define model list → run all → generate comparison table
4. **vLLM-accelerated eval** — 5-10× faster for large benchmarks

See [references/lm-evaluation-harness.md](references/lm-evaluation-harness.md) for the full guide including all workflows, troubleshooting, and advanced topics.

## Weights & Biases (W&B)

Full-lifecycle ML experiment tracking platform with 200,000+ practitioners and 100+ framework integrations.

**Capabilities:**
- Automatic metric logging with real-time dashboards
- Hyperparameter sweeps (grid, random, Bayesian)
- Artifact and model registry with lineage tracking
- Team collaboration with shared projects and reports
- Integrations: PyTorch, TensorFlow, HuggingFace, PyTorch Lightning, Keras

**Quick commands:**
| Action | Command |
|--------|---------|
| Install | `pip install wandb && wandb login` |
| Track experiment | `wandb.init(project="my-project", config={...})` |
| Log metrics | `wandb.log({"loss": loss, "accuracy": acc})` |
| Run sweep | `wandb.sweep(sweep_config, project="my-project")` |

**Key concepts:**
- **Projects** — collections of related experiments
- **Runs** — single training execution with config, metrics, artifacts
- **Sweeps** — automated hyperparameter optimization (bayes, grid, random)
- **Artifacts** — versioned datasets and models with lineage
- **Model Registry** — production-ready model versioning

See [references/weights-and-biases.md](references/weights-and-biases.md) for the full guide including PyTorch/HF/Lightning integrations, sweep strategies, artifacts, and team collaboration.

## When to Use

| Scenario | Use This |
|----------|----------|
| Benchmark a model on MMLU/GSM8K/HumanEval | lm-evaluation-harness |
| Compare multiple models on standard tasks | lm-evaluation-harness |
| Track training progress across checkpoints | lm-evaluation-harness (quick benchmarks) |
| Get reproducible academic metrics | lm-evaluation-harness |
| Log training metrics with real-time dashboards | Weights & Biases |
| Optimize hyperparameters with sweeps | Weights & Biases |
| Version datasets and models with lineage | Weights & Biases |
| Collaborate on ML experiments with a team | Weights & Biases |
| Evaluate an API-based model (OpenAI, Anthropic) | lm-evaluation-harness (API backend) |
| Need both benchmarking and tracking | Both — use lm-eval for standardized benchmarks, W&B for experiment lifecycle |

## Notes

- lm-evaluation-harness requires GPU for reasonable speed (CPU works but is very slow)
- W&B free tier: unlimited public projects, 100GB storage
- Use vLLM backend in lm-eval for 5-10× speedup
- W&B integrates natively with HuggingFace Trainer (`report_to="wandb"`)
- Both tools complement each other: use lm-eval for standardized benchmarks, W&B for the full experiment lifecycle
