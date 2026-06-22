---
name: ai-audio-generation
description: "AI audio generation: HeartMuLa (song from lyrics+tags, Suno-like) + AudioCraft/MusicGen (text-to-music, text-to-sound)."
version: 1.0.0
author: Hermes Agent (consolidated from heartmula + audiocraft-audio-generation)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [music, audio, generation, ai, text-to-music, text-to-sound, MusicGen, AudioGen, heartmula, Suno, lyrics, songs]
    category: media
    supersedes: [heartmula, audiocraft-audio-generation]
---

# AI Audio Generation

Generate music and audio from text prompts or lyrics. Two major open-source approaches:

| Engine | Input | Output | Best For |
|:-------|:------|:-------|:---------|
| **HeartMuLa** | Lyrics + tags | Full songs (vocal+instrumental) | Suno-like song generation with structure |
| **AudioCraft** (MusicGen/AudioGen) | Text description | Music or sound effects | Instrumental music, sound design, melody conditioning |

Choose HeartMuLa when you want structured songs with vocals. Choose AudioCraft when you want instrumental music from descriptions or need sound effects.

---

## Engine A: HeartMuLa — Song Generation from Lyrics + Tags

Open-source (Apache-2.0) music foundation model family. Comparable to Suno. Generates full songs with vocals conditioned on lyrics and style tags.

### Models in the Family
- **HeartMuLa** — Music language model (3B/7B) for generation from lyrics + tags
- **HeartCodec** — 12.5Hz music codec for high-fidelity audio reconstruction
- **HeartTranscriptor** — Whisper-based lyrics transcription
- **HeartCLAP** — Audio-text alignment model

### Hardware Requirements
- **Minimum**: 8GB VRAM with `--lazy_load true`
- **Recommended**: 16GB+ VRAM
- **No GPU?**: CPU mode works but extremely slow (~30-60 min/song). Recommend cloud GPU or https://heartmula.github.io/

### Installation

```bash
cd ~/
git clone https://github.com/HeartMuLa/heartlib.git
cd heartlib
uv venv --python 3.10 .venv
. .venv/bin/activate
uv pip install -e .

# Fix dependency conflicts (required as of Feb 2026)
uv pip install --upgrade datasets transformers
```

### Required Code Patches (for transformers 5.x)

**Patch 1 — RoPE cache fix** in `src/heartlib/heartmula/modeling_heartmula.py`:
After the `reset_caches` try/except in `setup_caches`, add:
```python
from torchtune.models.llama3_1._position_embeddings import Llama3ScaledRoPE
for module in self.modules():
    if isinstance(module, Llama3ScaledRoPE) and not module.is_cache_built:
        module.rope_init()
        module.to(device)
```

**Patch 2 — HeartCodec loading** in `src/heartlib/pipelines/music_generation.py`:
Add `ignore_mismatched_sizes=True` to ALL `HeartCodec.from_pretrained()` calls (2 locations).

### Download Models

```bash
cd heartlib
hf download --local-dir './ckpt' 'HeartMuLa/HeartMuLaGen'
hf download --local-dir './ckpt/HeartMuLa-oss-3B' 'HeartMuLa/HeartMuLa-oss-3B-happy-new-year'
hf download --local-dir './ckpt/HeartCodec-oss' 'HeartMuLa/HeartCodec-oss-20260123'
```

### Generation

```bash
cd heartlib
. .venv/bin/activate
python ./examples/run_music_generation.py \
  --model_path=./ckpt --version="3B" \
  --lyrics="./assets/lyrics.txt" --tags="./assets/tags.txt" \
  --save_path="./assets/output.mp3" --lazy_load true
```

**Tags** (comma-separated, no spaces): `piano,happy,wedding,synthesizer,romantic`

**Lyrics** (bracketed structure):
```
[Intro]
[Verse]
Your lyrics here...
[Chorus]
Chorus lyrics...
[Bridge]
[Outro]
```

**Key Parameters**: `--max_audio_length_ms` (240000), `--topk` (50), `--temperature` (1.0), `--cfg_scale` (1.5), `--lazy_load` (false)

**Performance**: RTF ≈ 1.0 (4-min song takes ~4 min). Output: MP3, 48kHz stereo, 128kbps.

### Pitfalls
1. Do NOT use bf16 for HeartCodec — degrades quality
2. Tags may be ignored (#90) — lyrics dominate
3. Triton not available on macOS
4. RTX 5080 incompatibility reported

### Songwriting Craft & Suno AI Music

See `references/songwriting-craft.md` for detailed songwriting guidance, Suno prompt engineering, metatags, parody/adaptation, and workflow.

### Links
- Repo: https://github.com/HeartMuLa/heartlib
- Models: https://huggingface.co/HeartMuLa
- Paper: https://arxiv.org/abs/2601.10547

---

## Engine B: AudioCraft — Meta's MusicGen & AudioGen

Text-to-music and text-to-sound generation via Meta's AudioCraft framework. Best for instrumental music, sound effects, melody conditioning, and style transfer.

### Model Variants

| Model | Size | Use Case |
|:------|:-----|:---------|
| `musicgen-small` | 300M | Quick generation |
| `musicgen-medium` | 1.5B | Balanced quality |
| `musicgen-large` | 3.3B | Best quality |
| `musicgen-melody` | 1.5B | Melody conditioning |
| `musicgen-stereo-*` | Varies | Stereo output |
| `musicgen-style` | 1.5B | Reference-based style transfer |
| `audiogen-medium` | 1.5B | Sound effects |

### Installation

```bash
pip install audiocraft
# or: pip install transformers torch torchaudio
```

### Quick Start

```python
import torchaudio
from audiocraft.models import MusicGen

model = MusicGen.get_pretrained('facebook/musicgen-small')
model.set_generation_params(duration=8, top_k=250, temperature=1.0)
wav = model.generate(["happy upbeat electronic dance music with synths"])
torchaudio.save("output.wav", wav[0].cpu(), sample_rate=32000)
```

### Key Capabilities
- **Text-to-music**: Describe the music, get audio
- **Melody conditioning**: Provide a melody reference, generate in that style
- **Stereo generation**: Full stereo with stereo model variants
- **Style transfer**: Generate matching a reference audio's style
- **Audio continuation**: Extend existing audio
- **Sound effects**: AudioGen for environmental sounds, Foley

### Generation Parameters
| Parameter | Default | Description |
|:-----------|:---------|:-------------|
| `duration` | 8.0 | Seconds (1-120) |
| `top_k` | 250 | Sampling diversity |
| `temperature` | 1.0 | Creativity |
| `cfg_coef` | 3.0 | Text adherence |

### GPU Memory (FP16)
| Model | VRAM |
|:------|:-----|
| small | ~2GB |
| medium | ~4GB |
| large | ~8GB |

### Common Issues
| Issue | Solution |
|:-------|:---------|
| CUDA OOM | Smaller model, reduce duration |
| Poor quality | Increase cfg_coef, better prompts |
| Audio artifacts | Try different temperature |

For full API reference with all code examples, see `references/audiocraft-api-reference.md`.
For advanced usage (training, fine-tuning, deployment), see `references/audiocraft-advanced-usage.md`.
For troubleshooting, see `references/audiocraft-troubleshooting.md`.

### Resources
- GitHub: https://github.com/facebookresearch/audiocraft
- MusicGen paper: https://arxiv.org/abs/2306.05284
- AudioGen paper: https://arxiv.org/abs/2209.15352
- HuggingFace: https://huggingface.co/facebook/musicgen-small

---

## References

| File | What |
|------|------|
| `references/songwriting-craft.md` | Song structure, rhyme, meter, Suno prompt engineering, metatags, parody |
| `references/audiocraft-api-reference.md` | Full AudioCraft code examples (MusicGen, AudioGen, EnCodec, workflows) |
| `references/audiocraft-advanced-usage.md` | Training, fine-tuning, deployment |
| `references/audiocraft-troubleshooting.md` | Common issues and solutions |
