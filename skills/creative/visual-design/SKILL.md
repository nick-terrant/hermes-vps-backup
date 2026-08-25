---
name: visual-design
description: "Visual content creation: HTML design artifacts (landing pages, prototypes, brand design systems, decks) and AI image generation (illustrations, comics, infographics)."
version: 1.0.0
author: Hermes Agent (consolidated from claude-design, baoyu-infographic)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [design, html, prototype, ux, ui, creative, artifact, deck, infographic, illustration, comic, image-generation, brand, design-system, design-tokens]
    supersedes: [claude-design, baoyu-infographic]
---

# Visual Design

Unified skill for visual content creation. Two production modes:

- **HTML Design Artifacts** — Design and build interactive HTML pages, prototypes, decks, and design systems
- **AI Image Generation** — Generate illustrations, comics, infographics, and data visualizations

**Pick your section:**
- [Section I: HTML Design Artifacts](#section-i-html-design-artifacts) — Landing pages, prototypes, brand systems, token specs
- [Section II: AI Image Generation](#section-ii-ai-image-generation) — Illustrations, comics, infographics

---

# Section I: HTML Design Artifacts

## What This Covers

| Mode | Description | When to use |
|------|-------------|-------------|
| **From-scratch design** | Design process and taste — scope a brief, produce variants, verify local HTML | Custom artifact with no specific brand |
| **Brand design systems** | 54 ready-to-paste design systems in `templates/html/` | "Make it look like Stripe/Linear/Vercel" |
| **Token spec files** | Google's DESIGN.md format — WCAG contrast, Tailwind/DTCG export | Formal design-token spec file |

## Runtime Mode

CLI/API mode — ignore hosted-only tool references (`done()`, `show_html()`, project panes). Deliverable: complete local HTML file.

## Brand Templates

54 brand design systems in `templates/html/` — each contains exact colors, typography, components, and CSS for a major brand:

| Brand | File | Brand | File |
|-------|------|-------|------|
| Stripe | `templates/html/stripe.md` | Linear | `templates/html/linear.md` |
| Vercel | `templates/html/vercel.md` | Notion | `templates/html/notion.md` |
| Airbnb | `templates/html/airbnb.md` | Claude | `templates/html/claude.md` |
| GitHub | `templates/html/github.md` | Arc | `templates/html/arc.md` |

See `templates/html/` for the full set of 54 brand templates.

## References

| File | Contents |
|------|----------|
| `references/html-design/*.md` | Design process guides |
| `scripts/design-md-starter.md` | Token spec starter template |

---

# Section II: AI Image Generation

Uses the `image_generate` tool (prompt-only, returns URL). All modes share core principles: data fidelity, style consistency, reproducible prompt files.

## Common Principles

- **Preserve source data faithfully** — never summarize or alter statistics
- **Strip secrets** — scan for API keys before including in output
- **Prompt files are mandatory** — save before generation
- **image_generate returns a URL** — download via `curl -o` before inserting local paths
- **Aspect ratios**: `landscape` (16:9), `portrait` (9:16), `square` (1:1)

### Section II-A: Article Illustration

Analyze articles, identify illustration positions, generate images with Type × Style × Palette consistency.

**Three Dimensions:**
- **Type**: infographic, scene, flowchart, comparison, framework, timeline
- **Style**: notion, warm, minimal, blueprint, watercolor, elegant (21 options in `references/image-gen/article-illustration/styles/`)
- **Palette**: macaron, warm, neon, mono-ink (in `references/image-gen/article-illustration/palettes/`)

**Workflow:** Analyze → Confirm settings → Generate outline → Generate prompts → Generate images → Insert into article

**References:** `references/image-gen/article-illustration/` (workflow, styles, palettes, prompt construction)

**System prompt:** `templates/baoyu/system.md`

### Section II-B: Knowledge Comics

Create educational comics with flexible art style × tone × layout combinations.

**Dimensions:**
- **Art**: ligne-claire, manga, realistic, ink-brush, chalk, minimalist
- **Tone**: neutral, warm, dramatic, romantic, energetic, vintage, action
- **Layout**: standard, cinematic, dense, splash, mixed, webtoon, four-panel

**Presets:** ohmsha (manga+neutral), wuxia (ink-brush+action), shoujo (manga+romantic), concept-story (manga+warm), four-panel (minimalist+neutral)

**Workflow:** Analyze → Confirm style+focus (required) → Storyboard+characters → Prompts → Pages

**References:** `references/image-gen/comics/` (art-styles, tones, layouts, presets, workflow, character templates)

### Section II-C: Infographics

Layout × Style combinations for information-dense visual summaries.

**21 Layouts** in `references/image-gen/layouts/`: bento-grid, linear-progression, binary-comparison, comparison-matrix, hierarchical-layers, tree-branching, hub-spoke, iceberg, funnel, dashboard, periodic-table, and more.

**21 Styles** in `references/image-gen/styles/`: craft-handmade, claymation, kawaii, cyberpunk-neon, bold-graphic, technical-schematic, pixel-art, and more.

Default: `bento-grid` + `craft-handmade`

**Workflow:** Analyze → Structured content → Recommend combinations → Confirm → Generate prompt → Generate image

## Pitfalls (All Modes)

1. **Data integrity is paramount** — never alter source statistics
2. **Strip secrets** before including in output files
3. **Prompt files are mandatory** — no image without saved prompt
4. **image_generate returns a URL** — download via `curl -o`
5. **Use absolute paths** for `curl -o`
6. **Comics: Step 2 confirmation required** — do not skip
7. **Comics: character consistency via text** — embed descriptions in page prompts
8. **Infographics: one message per section**

## Additional References

| File | Contents |
|------|----------|
| `references/image-gen/port-notes.md` | Port notes from upstream baoyu-skills |
