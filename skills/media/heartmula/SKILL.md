---
name: heartmula
description: "HeartMuLa: Suno-like song generation from lyrics + tags."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [music, audio, generation, ai, heartmula, heartcodec, lyrics, songs]
    related_skills: [audiocraft]
---

# HeartMuLa - Open-Source Music Generation

## Overview
HeartMuLa is a family of open-source music foundation models (Apache-2.0) that generates music conditioned on lyrics and tags, with multilingual support. Generates full songs from lyrics + tags. Comparable to Suno for open-source. Includes:
- **HeartMuLa** - Music language model (3B/7B) for generation from lyrics + tags
- **HeartCodec** - 12.5Hz music codec for high-fidelity audio reconstruction
- **HeartTranscriptor** - Whisper-based lyrics transcription
- **HeartCLAP** - Audio-text alignment model

## When to Use
- User wants to generate music/songs from text descriptions
- User wants an open-source Suno alternative
- User wants local/offline music generation
- User asks about HeartMuLa, heartlib, or AI music generation

## Hardware Requirements
- **Minimum**: 8GB VRAM with `--lazy_load true` (loads/unloads models sequentially)
- **Recommended**: 16GB+ VRAM for comfortable single-GPU usage
- **Multi-GPU**: Use `--mula_device cuda:0 --codec_device cuda:1` to split across GPUs
- 3B model with lazy_load peaks at ~6.2GB VRAM

## Installation Steps

### 1. Clone Repository
```bash
cd ~/  # or desired directory
git clone https://github.com/HeartMuLa/heartlib.git
cd heartlib
```

### 2. Create Virtual Environment (Python 3.10 required)
```bash
uv venv --python 3.10 .venv
. .venv/bin/activate
uv pip install -e .
```

### 3. Fix Dependency Compatibility Issues

**IMPORTANT**: As of Feb 2026, the pinned dependencies have conflicts with newer packages. Apply these fixes:

```bash
# Upgrade datasets (old version incompatible with current pyarrow)
uv pip install --upgrade datasets

# Upgrade transformers (needed for huggingface-hub 1.x compatibility)
uv pip install --upgrade transformers
```

### 4. Patch Source Code (Required for transformers 5.x)

**Patch 1 - RoPE cache fix** in `src/heartlib/heartmula/modeling_heartmula.py`:

In the `setup_caches` method of the `HeartMuLa` class, add RoPE reinitialization after the `reset_caches` try/except block and before the `with device:` block:

```python
# Re-initialize RoPE caches that were skipped during meta-device loading
from torchtune.models.llama3_1._position_embeddings import Llama3ScaledRoPE
for module in self.modules():
    if isinstance(module, Llama3ScaledRoPE) and not module.is_cache_built:
        module.rope_init()
        module.to(device)
```

**Why**: `from_pretrained` creates model on meta device first; `Llama3ScaledRoPE.rope_init()` skips cache building on meta tensors, then never rebuilds after weights are loaded to real device.

**Patch 2 - HeartCodec loading fix** in `src/heartlib/pipelines/music_generation.py`:

Add `ignore_mismatched_sizes=True` to ALL `HeartCodec.from_pretrained()` calls (there are 2: the eager load in `__init__` and the lazy load in the `codec` property).

**Why**: VQ codebook `initted` buffers have shape `[1]` in checkpoint vs `[]` in model. Same data, just scalar vs 0-d tensor. Safe to ignore.

### 5. Download Model Checkpoints
```bash
cd heartlib  # project root
hf download --local-dir './ckpt' 'HeartMuLa/HeartMuLaGen'
hf download --local-dir './ckpt/HeartMuLa-oss-3B' 'HeartMuLa/HeartMuLa-oss-3B-happy-new-year'
hf download --local-dir './ckpt/HeartCodec-oss' 'HeartMuLa/HeartCodec-oss-20260123'
```

All 3 can be downloaded in parallel. Total size is several GB.

## GPU / CUDA

HeartMuLa uses CUDA by default (`--mula_device cuda --codec_device cuda`). No extra setup needed if the user has an NVIDIA GPU with PyTorch CUDA support installed.

- The installed `torch==2.4.1` includes CUDA 12.1 support out of the box
- `torchtune` may report version `0.4.0+cpu` — this is just package metadata, it still uses CUDA via PyTorch
- To verify GPU is being used, look for "CUDA memory" lines in the output (e.g. "CUDA memory before unloading: 6.20 GB")
- **No GPU?** You can run on CPU with `--mula_device cpu --codec_device cpu`, but expect generation to be **extremely slow** (potentially 30-60+ minutes for a single song vs ~4 minutes on GPU). CPU mode also requires significant RAM (~12GB+ free). If the user has no NVIDIA GPU, recommend using a cloud GPU service (Google Colab free tier with T4, Lambda Labs, etc.) or the online demo at https://heartmula.github.io/ instead.

## Usage

### Basic Generation
```bash
cd heartlib
. .venv/bin/activate
python ./examples/run_music_generation.py \
  --model_path=./ckpt \
  --version="3B" \
  --lyrics="./assets/lyrics.txt" \
  --tags="./assets/tags.txt" \
  --save_path="./assets/output.mp3" \
  --lazy_load true
```

### Input Formatting

**Tags** (comma-separated, no spaces):
```
piano,happy,wedding,synthesizer,romantic
```
or
```
rock,energetic,guitar,drums,male-vocal
```

**Lyrics** (use bracketed structural tags):
```
[Intro]

[Verse]
Your lyrics here...

[Chorus]
Chorus lyrics...

[Bridge]
Bridge lyrics...

[Outro]
```

### Key Parameters
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--max_audio_length_ms` | 240000 | Max length in ms (240s = 4 min) |
| `--topk` | 50 | Top-k sampling |
| `--temperature` | 1.0 | Sampling temperature |
| `--cfg_scale` | 1.5 | Classifier-free guidance scale |
| `--lazy_load` | false | Load/unload models on demand (saves VRAM) |
| `--mula_dtype` | bfloat16 | Dtype for HeartMuLa (bf16 recommended) |
| `--codec_dtype` | float32 | Dtype for HeartCodec (fp32 recommended for quality) |

### Performance
- RTF (Real-Time Factor) ≈ 1.0 — a 4-minute song takes ~4 minutes to generate
- Output: MP3, 48kHz stereo, 128kbps

## Pitfalls
1. **Do NOT use bf16 for HeartCodec** — degrades audio quality. Use fp32 (default).
2. **Tags may be ignored** — known issue (#90). Lyrics tend to dominate; experiment with tag ordering.
3. **Triton not available on macOS** — Linux/CUDA only for GPU acceleration.
4. **RTX 5080 incompatibility** reported in upstream issues.
5. The dependency pin conflicts require the manual upgrades and patches described above.

## Songwriting Craft & Suno AI Music

HeartMuLa generates music from lyrics + tags locally. For cloud-based AI music generation (Suno) or songwriting craft guidance, use this section.

### Song Structure

Common skeletons (mix, modify, or invent):
- **ABABCB** — Verse/Chorus/Verse/Chorus/Bridge/Chorus (most pop/rock)
- **AABA** — Verse/Verse/Bridge/Verse (jazz standards, ballads)
- **ABAB** — Verse/Chorus alternating (simple, direct)
- **AAA** — Verse/Verse/Verse (folk, storytelling)

Building blocks: Intro, Verse, Pre-Chorus (optional), Chorus, Bridge, Outro.

### Rhyme & Meter

Rhyme types (tight to loose): perfect, family, assonance, consonance, near/slant. Mix them — all perfect rhymes sound nursery-rhyme-like; all slant sound lazy. Internal rhyme (rhyming within a line) adds richness.

Meter = rhythm of stressed vs unstressed syllables. Match syllable counts between parallel lines. Stressed syllables matter more than total count. Say it out loud — if you stumble, the meter needs work.

### Emotional Arc & Dynamics

Energy mapping (rough): Intro 2-3 → Verse 5-6 → Pre-Chorus 7 → Chorus 8-9 → Bridge varies → Final Chorus 9-10.

Key trick: **Contrast.** Whisper before a scream. Sparse before dense. Slow before fast. The drop works because of the buildup. Silence is an instrument.

### Lyrics That Work

- **Show, don't tell** (usually): "Your hoodie's still on the hook by the door" > "I was sad"
- **The hook**: line people remember — usually title or core phrase, placed where it lands hardest (first/last line of chorus)
- **Prosody**: stable feelings → settled melodies, perfect rhymes, resolved chords. Unstable feelings → wandering melodies, near-rhymes, unresolved chords
- Avoid: clichés on autopilot, Yoda-speak to force rhymes, same energy in every section

### Parody & Adaptation

Map the original's structure first: count syllables per line, mark rhyme scheme, identify stressed syllables, note held/sustained notes. Match stressed syllables to the same beats. Total syllable count can flex by 1-2 unstressed. On held notes, match the vowel sound. Keep some original lines intact for recognizability.

### Suno AI Prompt Engineering

**Style/Genre field formula**: Genre + Mood + Era + Instruments + Vocal Style + Production + Dynamics.

```
BAD:  "sad rock song"
GOOD: "Cinematic orchestral spy thriller, 1960s Cold War era, smoky
       sultry female vocalist, big band jazz, brass section,
       sweeping strings, minor key, vintage analog warmth"
```

Rules: No artist names or trademarks (describe the sound instead). Up to 1,000 chars in Style field (v4.5+). Describe the dynamic journey, not just the genre.

**Metatags** (in `[brackets]` inside lyrics field):
- Structure: `[Intro]` `[Verse]` `[Chorus]` `[Bridge]` `[Instrumental]` `[Outro]`
- Vocals: `[Whispered]` `[Belted]` `[Falsetto]` `[Harmonies]` `[Choir]`
- Dynamics: `[High Energy]` `[Building Energy]` `[Emotional Climax]` `[Quiet arrangement]`
- Atmosphere: `[Melancholic]` `[Euphoric]` `[Nostalgic]` `[Dreamy]`

Keep to 5-8 tags per section max. Don't contradict yourself. Put tags in BOTH style field and lyrics for reinforcement.

**Custom Mode**: Always use Custom Mode for serious work (separate Style + Lyrics fields). Lyrics field limit ~3,000 chars. Always add structural tags.

**Phonetic tricks**: Spell words as they SOUND ("through" → "thru"). ALL CAPS = louder. Vowel extension for sustained notes ("lo-o-o-ove"). Spell out numbers and space acronyms.

### Workflow

1. Write concept/hook first (emotional core)
2. If adapting, map original structure
3. Generate raw material freely before structuring
4. Draft lyrics into structure
5. Read/sing aloud — catch stumbles, fix meter
6. Build Suno style description (dynamic journey)
7. Add metatags for performance direction
8. Generate 3-5 variations minimum — ~3-5 generations per 1 good result

---

## Links
- Repo: https://github.com/HeartMuLa/heartlib
- Models: https://huggingface.co/HeartMuLa
- Paper: https://arxiv.org/abs/2601.10547
- License: Apache-2.0
