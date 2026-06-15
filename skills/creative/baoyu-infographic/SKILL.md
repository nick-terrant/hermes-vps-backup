---
name: baoyu-infographic
description: "Baoyu visual content generation: article illustrations, comics, infographics, and data visualizations."
version: 2.0.0
author: 宝玉 (JimLiu)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [infographic, illustration, comic, visual-summary, creative, image-generation, article-illustration, knowledge-comic]
    category: creative
    homepage: https://github.com/JimLiu/baoyu-skills#baoyu-infographic
---

# Baoyu Visual Content Generator

Umbrella skill for all Baoyu visual content generation: article illustrations, knowledge comics, infographics, and data visualizations. Adapted from [baoyu-skills](https://github.com/JimLiu/baoyu-skills) for Hermes Agent's tool ecosystem.

All three modes share the same underlying `image_generate` tool (prompt-only, returns URL) and the same core principles: data fidelity, style consistency, and reproducible prompt files.

## Common Principles

- **Preserve source data faithfully** — never summarize, paraphrase, or alter statistics
- **Strip secrets** — scan source content for API keys, tokens, or credentials before including in any output file
- **Prompt files are mandatory** — every image must have a saved prompt file before generation; the file is the reproducibility record
- **image_generate returns a URL** — always download via `curl -o` (use absolute paths) before inserting local paths
- **Aspect ratio mapping**: `16:9` → `landscape`, `9:16` → `portrait`, `1:1` → `square`; custom ratios → nearest named option
- **No backend selection from the agent** — `image_generate` uses whatever model the user configured

---

## Article Illustration

Analyze articles, identify illustration positions, generate images with **Type × Style × Palette** consistency.

### When to Use

Trigger when the user asks to illustrate an article, add images to an article, generate illustrations for content, or uses phrases like "为文章配图", "illustrate article", or "add images". The user provides an article (file path or pasted content) and optionally specifies type, style, palette, or density.

### Three Dimensions

| Dimension | Controls | Examples |
|-----------|----------|----------|
| **Type** | Information structure | infographic, scene, flowchart, comparison, framework, timeline |
| **Style** | Rendering approach | notion, warm, minimal, blueprint, watercolor, elegant |
| **Palette** | Color scheme (optional) | macaron, warm, neon — overrides style's default colors |

Combine freely: `type=infographic, style=vector-illustration, palette=macaron`. Or use presets: `edu-visual` → type + style + palette in one shot.

### Types

| Type | Best For |
|------|----------|
| `infographic` | Data, metrics, technical |
| `scene` | Narratives, emotional |
| `flowchart` | Processes, workflows |
| `comparison` | Side-by-side, options |
| `framework` | Models, architecture |
| `timeline` | History, evolution |

### Core Principles

- **Visualize concepts, not metaphors** — illustrate the underlying concept, not literal metaphors
- **Labels use article data** — actual numbers, terms, and quotes from the article
- **Prompt files are reproducibility records** — every illustration must have a saved prompt under `prompts/` before image generation

### Workflow

```
Step 1: Detect reference images (if provided) — vision_analyze → record traits in text
Step 2: Analyze content → analysis.md (content type, purpose, core arguments, positions)
Step 3: Confirm settings via clarify — Preset/Type, Density, Style, Palette, Language
Step 4: Generate outline → outline.md (frontmatter + entries per illustration)
Step 5: Generate prompts → prompts/NN-{type}-{slug}.md (BLOCKING: save prompt file before any image)
Step 6: Generate images → image_generate + curl download
Step 7: Finalize — insert image refs into article, report summary
```

### Output Structure

```
{output-dir}/
├── source-{slug}.{ext}    # Only for pasted content
├── outline.md
├── prompts/
│   └── NN-{type}-{slug}.md
└── NN-{type}-{slug}.png
```

Default output: `{article-dir}/imgs/` for file input, `illustrations/{topic-slug}/` for pasted content.

### References

| File | Content |
|------|---------|
| `references/article-illustration/workflow.md` | Detailed step-by-step procedures |
| `references/article-illustration/usage.md` | Invocation examples |
| `references/article-illustration/styles.md` | Style gallery + Palette gallery |
| `references/article-illustration/style-presets.md` | Preset shortcuts (type + style + palette) |
| `references/article-illustration/prompt-construction.md` | Prompt templates |
| `references/article-illustration/palettes/` | Palette definitions (macaron, mono-ink, neon, warm) |
| `references/article-illustration/styles/` | 21 style definitions |
| `prompts/system.md` | System prompt for article illustration |

---

## Comics

Create original knowledge comics (知识漫画) with flexible art style × tone combinations.

### When to Use

Trigger when the user asks to create a knowledge/educational comic, biography comic, tutorial comic, or uses terms like "知识漫画", "教育漫画", or "Logicomix-style". The user provides content (text, file path, URL, or topic) and optionally specifies art style, tone, layout, aspect ratio, or language.

### Reference Images

`image_generate` is prompt-only — when the user supplies reference images, extract traits in text via `vision_analyze` and embed them in every page prompt. Modes: `style`, `palette`, `scene`. Record traits in each page's prompt frontmatter.

Character consistency is driven by **text descriptions** in `characters/characters.md` embedded inline in every page prompt.

### Visual Dimensions

| Option | Values | Description |
|--------|--------|-------------|
| Art | ligne-claire (default), manga, realistic, ink-brush, chalk, minimalist | Art style / rendering |
| Tone | neutral (default), warm, dramatic, romantic, energetic, vintage, action | Mood / atmosphere |
| Layout | standard (default), cinematic, dense, splash, mixed, webtoon, four-panel | Panel arrangement |
| Aspect | 3:4 (default), 4:3, 16:9 | Page aspect ratio |

### Presets (art + tone + special rules)

| Preset | Equivalent | Hook |
|--------|-----------|------|
| `ohmsha` | manga + neutral | Visual metaphors, no talking heads |
| `wuxia` | ink-brush + action | Qi effects, combat visuals |
| `shoujo` | manga + romantic | Decorative, eye details |
| `concept-story` | manga + warm | Visual symbol system, growth arc |
| `four-panel` | minimalist + neutral + four-panel | 起承转結, B&W + spot color |

### Workflow

```
Step 1.1: Analyze content → analysis.md, source-{slug}.md
Step 1.2: Check existing directory → handle conflicts
Step 2: Confirm style + focus + audience + reviews (REQUIRED — use clarify)
Step 3: Generate storyboard + characters → storyboard.md, characters/
Step 4: Review outline (conditional)
Step 5: Generate prompts → prompts/*.md (with character descriptions embedded)
Step 6: Review prompts (conditional)
Step 7.1: Generate character sheet (if needed) → characters/characters.png
Step 7.2: Generate pages → *.png files (always use absolute curl -o paths)
Step 8: Completion report
```

**Critical**: Step 2 confirmation is required. Clarify timeout = default for that one question only, not all. Steps 4/6 are conditional (only if user requested).

### Output Structure

```
comic/{topic-slug}/
├── source-{slug}.md
├── analysis.md
├── storyboard.md
├── characters/characters.md
├── characters/characters.png
├── prompts/NN-{cover|page}-[slug].md
├── NN-{cover|page}-[slug].png
└── refs/NN-ref-{slug}.{ext}  (optional, provenance)
```

### References

**Core Templates**:
- `references/comics/analysis-framework.md` — Deep content analysis
- `references/comics/character-template.md` — Character definition format
- `references/comics/storyboard-template.md` — Storyboard structure
- `references/comics/ohmsha-guide.md` — Ohmsha manga specifics

**Style Definitions**:
- `references/comics/art-styles/` — 6 art styles (ligne-claire, manga, realistic, ink-brush, chalk, minimalist)
- `references/comics/tones/` — 7 tones (neutral, warm, dramatic, romantic, energetic, vintage, action)
- `references/comics/presets/` — 5 presets with special rules (ohmsha, wuxia, shoujo, concept-story, four-panel)
- `references/comics/layouts/` — 7 layouts (standard, cinematic, dense, splash, mixed, webtoon, four-panel)

**Workflow**:
- `references/comics/workflow.md` — Full step-by-step workflow
- `references/comics/auto-selection.md` — Content signal → preset mapping
- `references/comics/partial-workflows.md` — Partial workflow options (storyboard only, prompts only, etc.)

---

## Infographics

Two dimensions: **layout** (information structure) × **style** (visual aesthetics). Freely combine any layout with any style.

### When to Use

Trigger when the user asks to create an infographic, visual summary, information graphic, or uses terms like "信息图", "可视化", or "高密度信息大图". The user provides content (text, file path, URL, or topic) and optionally specifies layout, style, aspect ratio, or language.

### Options

| Option | Values |
|--------|--------|
| Layout | 21 options (see Layout Gallery), default: bento-grid |
| Style | 21 options (see Style Gallery), default: craft-handmade |
| Aspect | landscape (16:9), portrait (9:16), square (1:1), or custom W:H |
| Language | en, zh, ja, etc. |

### Layout Gallery

| Layout | Best For |
|--------|----------|
| `linear-progression` | Timelines, processes, tutorials |
| `binary-comparison` | A vs B, before-after, pros-cons |
| `comparison-matrix` | Multi-factor comparisons |
| `hierarchical-layers` | Pyramids, priority levels |
| `tree-branching` | Categories, taxonomies |
| `hub-spoke` | Central concept with related items |
| `structural-breakdown` | Exploded views, cross-sections |
| `bento-grid` | Multiple topics, overview (default) |
| `iceberg` | Surface vs hidden aspects |
| `bridge` | Problem-solution |
| `funnel` | Conversion, filtering |
| `isometric-map` | Spatial relationships |
| `dashboard` | Metrics, KPIs |
| `periodic-table` | Categorized collections |
| `comic-strip` | Narratives, sequences |
| `story-mountain` | Plot structure, tension arcs |
| `jigsaw` | Interconnected parts |
| `venn-diagram` | Overlapping concepts |
| `winding-roadmap` | Journey, milestones |
| `circular-flow` | Cycles, recurring processes |
| `dense-modules` | High-density modules, data-rich guides |

Full definitions: `references/layouts/<layout>.md`

### Style Gallery

| Style | Description |
|-------|-------------|
| `craft-handmade` | Hand-drawn, paper craft (default) |
| `claymation` | 3D clay figures, stop-motion |
| `kawaii` | Japanese cute, pastels |
| `storybook-watercolor` | Soft painted, whimsical |
| `chalkboard` | Chalk on black board |
| `cyberpunk-neon` | Neon glow, futuristic |
| `bold-graphic` | Comic style, halftone |
| `aged-academia` | Vintage science, sepia |
| `corporate-memphis` | Flat vector, vibrant |
| `technical-schematic` | Blueprint, engineering |
| `origami` | Folded paper, geometric |
| `pixel-art` | Retro 8-bit |
| `ui-wireframe` | Grayscale interface mockup |
| `subway-map` | Transit diagram |
| `ikea-manual` | Minimal line art |
| `knolling` | Organized flat-lay |
| `lego-brick` | Toy brick construction |
| `pop-laboratory` | Blueprint grid, lab precision |
| `morandi-journal` | Hand-drawn doodle, warm Morandi tones |
| `retro-pop-grid` | 1970s retro pop art, Swiss grid |
| `hand-drawn-edu` | Macaron pastels, hand-drawn wobble |

Full definitions: `references/styles/<style>.md`

### Recommended Combinations

| Content Type | Layout + Style |
|--------------|----------------|
| Timeline/History | `linear-progression` + `craft-handmade` |
| Step-by-step | `linear-progression` + `ikea-manual` |
| A vs B | `binary-comparison` + `corporate-memphis` |
| Hierarchy | `hierarchical-layers` + `craft-handmade` |
| Overlap | `venn-diagram` + `craft-handmade` |
| Conversion | `funnel` + `corporate-memphis` |
| Cycles | `circular-flow` + `craft-handmade` |
| Technical | `structural-breakdown` + `technical-schematic` |
| Metrics | `dashboard` + `corporate-memphis` |
| Educational | `bento-grid` + `chalkboard` |
| Journey | `winding-roadmap` + `storybook-watercolor` |
| Categories | `periodic-table` + `bold-graphic` |
| Product Guide | `dense-modules` + `morandi-journal` |
| Technical Guide | `dense-modules` + `pop-laboratory` |
| Trendy Guide | `dense-modules` + `retro-pop-grid` |

Default: `bento-grid` + `craft-handmade`

### Keyword Shortcuts

| User Keyword | Layout | Recommended Styles | Default Aspect | Prompt Notes |
|--------------|--------|--------------------|----------------|--------------|
| 高密度信息大图 / high-density-info | `dense-modules` | `morandi-journal`, `pop-laboratory`, `retro-pop-grid` | portrait | — |
| 信息图 / infographic | `bento-grid` | `craft-handmade` | landscape | Minimalist: clean canvas, ample whitespace, no complex background textures. |

### Workflow

```
Step 1: Analyze content → source.md, analysis.md
Step 2: Generate structured content → structured-content.md
Step 3: Recommend 3-5 layout×style combinations (check keyword shortcuts first)
Step 4: Confirm options via clarify — combination, aspect, language
Step 5: Generate prompt → prompts/infographic.md (load layout + style refs)
Step 6: Generate image → image_generate + curl download
Step 7: Output summary
```

### Output Structure

```
infographic/{topic-slug}/
├── source-{slug}.{ext}
├── analysis.md
├── structured-content.md
├── prompts/infographic.md
└── infographic.png
```

Slug: 2-4 words kebab-case from topic. Conflict: append `-YYYYMMDD-HHMMSS`.

### References

- `references/analysis-framework.md` — Analysis methodology
- `references/structured-content-template.md` — Content format
- `references/base-prompt.md` — Prompt template
- `references/layouts/<layout>.md` — 21 layout definitions
- `references/styles/<style>.md` — 21 style definitions

---

## Pitfalls (All Modes)

1. **Data integrity is paramount** — never summarize, paraphrase, or alter source statistics. "73% increase" stays "73% increase".
2. **Strip secrets** — scan source content for API keys, tokens, or credentials before including in any output file.
3. **Prompt files are mandatory** — no image generation without a saved prompt file.
4. **image_generate returns a URL, not a file** — always download via `curl` before inserting local paths.
5. **Use absolute paths for curl -o** — never rely on persistent-shell CWD across batches.
6. **image_generate aspect ratios** — the tool only supports `landscape`, `portrait`, and `square`.
7. **No backend selection from the agent** — `image_generate` uses whatever model the user configured.
8. **Comics: Step 2 confirmation required** — do not skip; clarify timeout defaults one question only.
9. **Comics: character consistency via text** — page prompts embed text descriptions from characters.md, not the PNG sheet.
10. **Infographics: one message per section** — each section should convey one clear concept.
