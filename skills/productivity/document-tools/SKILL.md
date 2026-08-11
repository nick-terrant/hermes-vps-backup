---
name: document-tools
description: "Document creation and extraction: OCR, PDF extraction, YouTube transcripts, and PowerPoint manipulation."
version: 1.0.0
author: Hermes Agent (consolidated from ocr-and-documents, powerpoint)
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  commands: [python3]
metadata:
  hermes:
    tags: [OCR, PDF, PowerPoint, Documents, Extraction, YouTube, Transcripts, Office, pptx, PyMuPDF]
    supersedes: [ocr-and-documents, powerpoint]
---

# Document Tools

Create, read, edit, and extract content from documents. Covers OCR, PDF extraction, YouTube transcript retrieval, and PowerPoint (.pptx) creation and editing.

**Pick your section:**
- [Section I: OCR & Document Extraction](#section-i-ocr-and-document-extraction) — PDF text/image extraction, OCR, YouTube transcripts
- [Section II: PowerPoint](#section-ii-powerpoint-pptx) — Create, read, edit .pptx presentations

---

# Section I: OCR & Document Extraction

Extract text from PDFs, images, and YouTube videos. Multiple extraction strategies depending on source type.

## Quick Reference — Which Tool for Which Source

| Source | Method | Command |
|--------|--------|---------|
| PDF (text) | PyMuPDF | `scripts/extract_pymupdf.py file.pdf` |
| PDF (scanned/image) | Vision model | `vision_analyze(image_url=...)` |
| YouTube video | Transcript fetch | `scripts/fetch_transcript.py URL` |
| Markdown marker | Marker extraction | `scripts/extract_marker.py file.pdf` |
| Image (local) | Vision model | `vision_analyze(image_url="path/to/img")` |
| Image (URL) | Vision model | `vision_analyze(image_url="https://...")` |

## PDF Extraction with PyMuPDF

```bash
# Extract all text from a PDF
python3 scripts/extract_pymupdf.py document.pdf

# Extract specific pages (1-indexed)
python3 scripts/extract_pymupdf.py document.pdf --pages 1-5

# Extract with images saved alongside
python3 scripts/extract_pymupdf.py document.pdf --extract-images --output-dir ./output
```

The script outputs Markdown-formatted text with page separators.

## YouTube Transcript Extraction

```bash
# Get transcript as text
python3 scripts/fetch_transcript.py "https://youtube.com/watch?v=VIDEO_ID"

# Get structured output (timestamps + text)
python3 scripts/fetch_transcript.py "https://youtube.com/watch?v=VIDEO_ID" --structured

# Available output formats: text, srt, json, markdown
python3 scripts/fetch_transcript.py "https://youtube.com/watch?v=VIDEO_ID" --format srt
```

## OCR with Marker

For scanned PDFs or images where text layer is absent:

```bash
# Extract with OCR (requires marker package)
python3 scripts/extract_marker.py scanned_document.pdf

# High-quality OCR mode
python3 scripts/extract_marker.py scanned_document.pdf --quality high
```

## Vision-Based OCR

For images or PDFs rendered as images where traditional OCR struggles:

```
# Use vision_analyze tool directly on the image
vision_analyze(image_url="path/to/image.png", question="Extract all text from this image")
```

## YouTube Output Format Reference

See `references/youtube-output-formats.md` for details on transcript formats (SRT, VTT, JSON, plain text).

## References

| File | Contents |
|------|----------|
| `references/youtube-output-formats.md` | Transcript format specifications |
| `scripts/extract_pymupdf.py` | PyMuPDF-based PDF text extraction |
| `scripts/extract_marker.py` | Marker-based OCR extraction |
| `scripts/fetch_transcript.py` | YouTube transcript fetcher |

---

# Section II: PowerPoint (.pptx)

Create, read, edit, and manipulate PowerPoint presentations programmatically.

## Quick Start

```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[6])  # Blank layout

# Add text box
txBox = slide.shapes.add_textbox(Inches(1), Inches(1), Inches(8), Inches(4))
tf = txBox.text_frame
tf.word_wrap = True
p = tf.paragraphs[0]
p.text = "Hello, World!"
p.font.size = Pt(36)
p.font.bold = True

prs.save("output.pptx")
```

## Key Operations

### Creating Presentations

```python
from pptx import Presentation

prs = Presentation()  # Default 16:9
# Or specify slide size:
prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)

slide = prs.slides.add_slide(prs.slide_layouts[6])  # 6 = Blank
prs.save("deck.pptx")
```

### Working with Text

```python
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor

txBox = slide.shapes.add_textbox(Inches(1), Inches(1), Inches(8), Inches(2))
tf = txBox.text_frame
tf.word_wrap = True

# First paragraph
p = tf.paragraphs[0]
p.text = "Title"
p.font.size = Pt(44)
p.font.color.rgb = RGBColor(0x1A, 0x1A, 0x2E)
p.font.bold = True

# Add more paragraphs
p = tf.add_paragraph()
p.text = "Subtitle text"
p.font.size = Pt(24)
p.space_before = Pt(12)
```

### Adding Shapes & Images

```python
from pptx.enum.shapes import MSO_SHAPE

# Rectangle
shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, Inches(2), Inches(2), Inches(4), Inches(3))
shape.fill.solid()
shape.fill.fore_color.rgb = RGBColor(0x00, 0x7A, 0xCC)

# Image
slide.shapes.add_picture("image.png", Inches(1), Inches(1), width=Inches(6))

# Table
rows, cols = 3, 4
table_shape = slide.shapes.add_table(rows, cols, Inches(1), Inches(4), Inches(8), Inches(3))
table = table_shape.table
table.cell(0, 0).text = "Header 1"
```

### Reading Presentations

```python
prs = Presentation("existing.pptx")
for slide in prs.slides:
    for shape in slide.shapes:
        if shape.has_text_frame:
            for paragraph in shape.text_frame.paragraphs:
                print(paragraph.text)
        if shape.has_table:
            for row in shape.table.rows:
                for cell in row.cells:
                    print(cell.text)
```

## Editing Existing Presentations

The `scripts/clean.py` utility strips unused content:

```bash
python3 scripts/clean.py presentation.pptx --output cleaned.pptx
```

## Office XML Schemas

The `scripts/office/` directory contains OOXML schema definitions for advanced manipulation:

| Schema | Purpose |
|--------|---------|
| `shared-documentProperties*.xsd` | Document metadata |
| `dml-main.xsd` | Drawing markup language |
| `pml.xsd` | Presentation markup |
| `wml.xsd` | Word processing markup |
| `vml-main.xsd` | Vector markup language |

## Advanced: Editing Markup

For complex edits that python-pptx doesn't support, work directly with the OOXML:

```python
import zipfile, re
# .pptx files are ZIP archives containing XML
# Modify slide XML directly, then repack
```

See `editing.md` and `pptxgenjs.md` for additional patterns including programmatic slide generation with pptxgenjs.

## References

| File | Contents |
|------|----------|
| `references/pptxgenjs.md` | pptxgenjs library reference (Node.js alternative) |
| `editing.md` | Direct XML editing techniques |
| `scripts/clean.py` | Presentation cleanup utility |
| `scripts/office/` | OOXML schema definitions for advanced manipulation |
