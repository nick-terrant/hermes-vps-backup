---
name: programmatic-animation
description: "Programmatic animation: Manim CE for math/algo videos, p5.js for generative art and interactive sketches."
version: 1.0.0
author: Hermes Agent (consolidated from manim-video, p5js)
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  commands: [manim]
metadata:
  hermes:
    tags: [Manim, p5.js, Animation, Generative Art, Math Visualization, WebGL, 3Blue1Brown, Creative Coding]
    supersedes: [manim-video, p5js]
---

# Programmatic Animation

Create animations and visual art programmatically. Two frameworks, one skill:

- **Manim CE** — 3Blue1Brown-style math and algorithm explainer videos (Python)
- **p5.js** — Generative art, interactive sketches, WebGL experiments (JavaScript)

**Pick your section:**
- [Section I: Manim CE](#section-i-manim-ce-math-and-algorithm-videos) — Python-based math animation
- [Section II: p5.js](#section-ii-p5js-generative-art-and-interactive-sketches) — JavaScript creative coding

---

# Section I: Manim CE (Math and Algorithm Videos)

## Setup

```bash
pip install manim            # or: pip install manim[interactive]
# Verify
manim --version
manim --help

# LaTeX (for math rendering):
#   macOS: brew install --cask mactex-no-gui
#   Linux: sudo apt install texlive-full (or texlive-latex-base for minimal)
#   Or use --format=mp4 with no LaTeX for text-only scenes
```

## Quick Start

```python
from manim import *

class HelloCircle(Scene):
    def construct(self):
        circle = Circle(color=BLUE)
        self.play(Create(circle))
        self.play(circle.animate.shift(RIGHT * 2))
        self.play(circle.animate.scale(0.5))
```

```bash
manim -pql scene.py HelloCircle   # Low quality preview (fast)
manim -pqh scene.py HelloCircle   # High quality (slow)
```

## Core Workflow

1. **Write scene class** in a `.py` file
2. **Preview:** `manim -pql file.py SceneName` (low quality, fast)
3. **Render:** `manim -pqh file.py SceneName` (high quality)
4. **Export options:** `-p` auto-play, `-s` save last frame as PNG

## Key Animation Methods

| Method | Effect |
|--------|--------|
| `self.play(Create(obj))` | Draw an object |
| `self.play(FadeIn(obj))` | Fade in |
| `self.play(Write(text))` | Write text stroke by stroke |
| `self.play(Transform(a, b))` | Morph one object into another |
| `self.play(obj.animate.shift(dir))` | Animate to new position |
| `self.play(obj.animate.scale(factor))` | Animate scaling |
| `self.play(Rotate(obj, angle))` | Rotate an object |
| `self.play(Indicate(obj))` | Flash/highlight |
| `self.play(FocusOn(obj))` | Camera focus zoom |
| `self.play(*[Create(m) for m in objects])` | Parallel creation |

## Mobjects (Mathematical Objects)

| Type | Class | Notes |
|------|-------|-------|
| Shapes | `Circle`, `Square`, `Rectangle`, `Polygon`, `Triangle` | `color=`, `fill_opacity=` |
| Lines | `Line`, `Arrow`, `DoubleArrow`, `Vector` | `start=`, `end=` |
| Text | `Text`, `Title`, `MathTex`, `Tex` | `MathTex(r"\frac{a}{b}")` |
| Graphs | `NumberPlane`, `Axes`, `CoordinateSystem` | Built-in axes with labels |
| Functions | `FunctionGraph`, `ParametricFunction` | Plot any callable |
| Groups | `VGroup`, `Group` | Treat multiple objects as one |
| Tables | `Table`, `MathTable` | `Table([["A","B"]], ...)` |

## Best Practices

1. **Start with `self.wait(1)`** at end of each conceptual section — creates pacing
2. **Use `VGroup`** to coordinate related objects
3. **`self.next_section()`** to organize output videos into chapters
4. **`self.play(AnimationGroup(..., lag_ratio=0.5))`** for staggered animations
5. **Prefer `animate` syntax** over `ApplyMethod` — cleaner and composable

## Render Targets

```bash
manim -ql  file.py Scene   # 480p, low quality
manim -qm  file.py Scene   # 720p, medium
manim -qh  file.py Scene   # 1080p, high
manim -qk  file.py Scene   # 2160p, 4K
manim -ql  file.py Scene -w  # White background (default black)
```

## References

| File | Contents |
|------|----------|
| `references/manim-animations.md` | Complete animation method reference |
| `references/manim-mobjects.md` | All mobject types with examples |
| `references/manim-camera-and-3d.md` | MovingCameraFrame, ThreeDScene, OpenGL |
| `references/manim-equations.md` | Math typesetting, LaTeX patterns |
| `references/manim-graphs-and-data.md` | Plots, charts, bar graphs |
| `references/manim-decoration.md` | SurroundingRect, BackgroundColoredVMobject |
| `references/manim-updaters-and-trackers.md` | `always_redraw`, ValueTracker |
| `references/manim-rendering.md` | Custom render targets, transparent BG |
| `references/manim-troubleshooting.md` | LaTeX errors, import issues, performance |
| `references/manim-visual-design.md` | Color, typography, composition |
| `references/manim-animation-design-thinking.md` | Pedagogical animation principles |
| `references/manim-scene-planning.md` | Scripting and storyboarding approach |
| `references/manim-paper-explainer.md` | Academic paper → animation workflow |
| `scripts/manim-setup.sh` | Environment setup script |

---

# Section II: p5.js (Generative Art and Interactive Sketches)

## Setup

```bash
# p5.js runs in the browser — no install needed
# Create an HTML file that loads p5.js from CDN, or use the serve script:

# Setup helper (creates project structure)
bash scripts/p5js-setup.sh my-sketch

# Serve a sketch locally
bash scripts/p5js-serve.sh my-sketch

# Export frames to PNG sequence
node scripts/p5js-export-frames.js my-sketch --output frames/ --count 300 --fps 30
```

## Quick Start

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
</head>
<body>
  <script>
    function setup() {
      createCanvas(800, 600);
    }
    function draw() {
      background(220);
      ellipse(mouseX, mouseY, 50, 50);
    }
  </script>
</body>
</html>
```

## Core Concepts

**Two mandatory functions:**
- `setup()` — Runs once. Create canvas, load assets, set initial state.
- `draw()` — Runs every frame (60fps by default). Put animation logic here.

**Coordinate system:** Origin at top-left. X increases right, Y increases down.

## Key p5.js Functions

| Category | Functions |
|----------|-----------|
| **Shape** | `rect()`, `ellipse()`, `triangle()`, `line()`, `quad()`, `polygon()` |
| **Color** | `fill()`, `stroke()`, `noFill()`, `noStroke()`, `background()`, `color()` |
| **Transform** | `translate()`, `rotate()`, `scale()`, `push()`, `pop()`, `applyMatrix()` |
| **Typography** | `text()`, `textSize()`, `textFont()`, `textAlign()`, `textWidth()` |
| **Math** | `random()`, `noise()`, `map()`, `constrain()`, `lerp()`, `dist()` |
| **Input** | `mouseX`, `mouseY`, `mouseIsPressed`, `keyIsDown()`, `mousePressed()` |
| **Array** | `createVector()`, `p5.Vector` class for 2D/3D math |

## Advanced Topics

### Perlin Noise
```javascript
let t = 0;
function draw() {
  let n = noise(t);
  ellipse(map(n, 0, 1, 0, width), height/2, 10, 10);
  t += 0.01;
}
```

### Particle Systems
```javascript
class Particle {
  constructor() {
    this.pos = createVector(random(width), random(height));
    this.vel = createVector(0, random(-2, -0.5));
    this.alpha = 255;
  }
  update() { this.pos.add(this.vel); this.alpha -= 3; }
  show() { noStroke(); fill(255, this.alpha); ellipse(this.pos.x, this.pos.y, 4); }
}
```

### Shaders (WEBGL)
```javascript
function setup() { createCanvas(800, 600, WEBGL); }
function draw() {
  background(0);
  // Use shader() with custom vertex/fragment shaders
  // Or use built-in: box(), sphere(), torus(), cylinder()
  rotateX(frameCount * 0.01);
  rotateY(frameCount * 0.01);
  box(200);
}
```

## Export Pipeline

```bash
# Export frames to PNG sequence
node scripts/p5js-export-frames.js sketch-dir --output frames/ --count 300 --fps 30

# Convert to video (requires ffmpeg)
ffmpeg -framerate 30 -i frames/frame_%04d.png -c:v libx264 -pix_fmt yuv420p output.mp4

# Or use the render script
bash scripts/p5js-render.sh sketch-dir --format mp4 --fps 30
```

## Viewer Template

See `templates/viewer.html` for a ready-made HTML wrapper that serves p5.js sketches with a clean UI.

## References

| File | Contents |
|------|----------|
| `references/p5js-core-api.md` | Complete p5.js function reference |
| `references/p5js-shapes-and-geometry.md` | Shape primitives, custom shapes, curves |
| `references/p5js-color-systems.md` | HSB, RGB, alpha, colorMode |
| `references/p5js-animation.md` | Frame-based animation, easing, state machines |
| `references/p5js-interaction.md` | Mouse, keyboard, touch, game-like input |
| `references/p5js-visual-effects.md` | Particles, trails, bloom, post-processing |
| `references/p5js-typography.md` | Text rendering, web fonts, text layout |
| `references/p5js-webgl-and-3d.md` | WEBGL mode, 3D primitives, shaders, cameras |
| `references/p5js-export-pipeline.md` | Frame export, video rendering, GIF creation |
| `references/p5js-troubleshooting.md` | Common issues and fixes |
| `templates/viewer.html` | HTML wrapper for serving p5.js sketches |
| `scripts/p5js-setup.sh` | Project scaffolding |
| `scripts/p5js-serve.sh` | Local HTTP server for sketches |
| `scripts/p5js-render.sh` | Render to video/GIF |
| `scripts/p5js-export-frames.js` | Frame-by-frame PNG export |
