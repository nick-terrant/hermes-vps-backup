---
name: diagramming
description: "Diagrams: SVG architecture diagrams (HTML) and hand-drawn Excalidraw JSON diagrams."
version: 1.0.0
author: Hermes Agent (consolidated from architecture-diagram + excalidraw)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [diagrams, architecture, SVG, HTML, Excalidraw, visualization, infrastructure, cloud, flowcharts, sequence-diagrams]
    category: creative
    supersedes: [architecture-diagram, excalidraw]
---

# Diagramming

Create technical diagrams in two formats. Choose based on use case:

| Need | Format | When |
|:-----|:-------|:----|
| Polished, dark-themed tech diagrams (cloud infra, microservices, deployment) | **SVG HTML** | Presentations, docs, README headers — opens in any browser |
| Hand-drawn whiteboard sketches (flowcharts, concept maps, sequence diagrams) | **Excalidraw JSON** | Quick sketches, collaborative editing at excalidraw.com |
| Editable by non-developers | **Excalidraw** | Shareable links, no-code editing |
| Print-quality, pixel-perfect | **SVG HTML** | Embedded in docs, PDF-ready |

---

## Format A: SVG Architecture Diagrams (HTML)

Generate professional, dark-themed technical architecture diagrams as standalone HTML files with inline SVG graphics. No external tools, no API keys, no rendering libraries — just write the HTML file and open it in a browser.

### Scope

**Best suited for:**
- Software system architecture (frontend / backend / database layers)
- Cloud infrastructure (VPC, regions, subnets, managed services)
- Microservice / service-mesh topology
- Database + API map, deployment diagrams

**Look elsewhere first for:**
- Physics, chemistry, math, biology, or other scientific subjects
- Physical objects (vehicles, hardware, anatomy, cross-sections)
- Floor plans, narrative journeys, educational / textbook-style visuals
- Hand-drawn whiteboard sketches → use Format B (Excalidraw)
- Animated explainers → use an animation skill

### Workflow

1. User describes their system architecture (components, connections, technologies)
2. Generate the HTML file following the design system below
3. Save with `write_file` to a `.html` file
4. User opens in any browser — works offline, no dependencies

### Design System

**Color Palette (Semantic Mapping):**

| Component Type | Fill (rgba) | Stroke (Hex) |
|:---|:---|:---|
| Frontend | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan-400) |
| Backend | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald-400) |
| Database | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet-400) |
| AWS/Cloud | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber-400) |
| Security | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose-400) |
| Message Bus | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange-400) |
| External | `rgba(30, 41, 59, 0.5)` | `#94a3b8` (slate-400) |

**Typography:** JetBrains Mono (Monospace), 12px / 9px / 8px / 7px. Background: Slate-950 (`#020617`) with 40px grid.

**Key Rules:**
- Components are rounded rectangles (`rx="6"`) with 1.5px strokes
- Use double-rect masking (opaque bg + semi-transparent styled) to prevent arrow bleed-through
- Arrows render behind components (draw early in SVG)
- Security flows: dashed lines in rose color
- Legend placement: outside all boundary boxes, 20px below lowest boundary
- Output: single `.html` file, no JavaScript, all CSS/SVG inline

### Template

Load the full HTML template:
```
skill_view(name="diagramming", file_path="templates/architecture-template.html")
```

---

## Format B: Excalidraw JSON Diagrams

Create hand-drawn diagrams by writing standard Excalidraw element JSON. Files open at [excalidraw.com](https://excalidraw.com) or can be uploaded for shareable links.

### When to use

Architecture diagrams, flowcharts, sequence diagrams, concept maps, wireframes — anything that benefits from a hand-drawn, approachable aesthetic. Prefer over SVG when you need collaborative editing or shareable links.

### Workflow

1. Write the elements JSON — an array of Excalidraw element objects
2. Save with `write_file` as a `.excalidraw` file
3. Optionally upload for a shareable link: `python scripts/upload.py path.excalidraw`

### File Format

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "hermes-agent",
  "elements": [ ...elements... ],
  "appState": { "viewBackgroundColor": "#ffffff" }
}
```

### Element Types

| Type | Key Fields | Notes |
|:-----|:-----------|:------|
| `rectangle` | `roundness: { "type": 3 }` for rounded | Add `boundElements` for labels |
| `ellipse` | Standard x/y/width/height | |
| `diamond` | Decision nodes | |
| `arrow` | `points`, `endArrowhead` | Use `startBinding`/`endBinding` to connect shapes |
| `text` | `containerId` for shape labels, standalone for annotations | Always set `fontFamily: 1` |

**Critical:** Do NOT use `"label": { "text": "..." }` on shapes. Use the container binding approach with `boundElements` + `containerId`.

### Drawing Order (z-order)

Array order = z-order (first = back, last = front). Emit progressively: background zones → shape → its bound text → its arrows → next shape. Always place bound text immediately after its container shape.

### Sizing

- Min `fontSize`: 16 (body), 20 (titles), never below 14
- Min shape size: 120×60
- 20-30px gaps between elements

### Color Palette

| Use | Fill | Hex |
|:-----|:------|:----|
| Primary / Input | Light Blue | `#a5d8ff` |
| Success / Output | Light Green | `#b2f2bb` |
| Warning / External | Light Orange | `#ffd8a8` |
| Processing / Special | Light Purple | `#d0bfff` |
| Error / Critical | Light Red | `#ffc9c9` |
| Notes / Decisions | Light Yellow | `#fff3bf` |
| Storage / Data | Light Teal | `#c3fae8` |

**Text contrast:** Never use light gray on white backgrounds. Min text color: `#757575`.

### Tips
- Do NOT use emoji in text — they don't render in Excalidraw's font
- For dark mode diagrams, see `references/excalidraw-dark-mode.md`
- For larger examples, see `references/excalidraw-examples.md`
- For full color tables, see `references/excalidraw-colors.md`

---

## References

| File | What |
|------|------|
| `templates/architecture-template.html` | Full SVG architecture diagram template with examples |
| `references/excalidraw-examples.md` | Larger Excalidraw diagram examples |
| `references/excalidraw-dark-mode.md` | Dark-mode Excalidraw diagram instructions |
| `references/excalidraw-colors.md` | Full Excalidraw color reference tables |
| `scripts/excalidraw-upload.py` | Upload .excalidraw to excalidraw.com for shareable link |
