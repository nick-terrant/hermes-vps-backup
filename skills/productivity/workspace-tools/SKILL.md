---
name: workspace-tools
description: "Workspace productivity: Google Workspace (Gmail, Calendar, Drive, Sheets, Docs), SaaS APIs (Airtable, Linear, Notion), and document tools (OCR, PDF, PowerPoint, YouTube transcripts)."
version: 1.0.0
author: Hermes Agent (consolidated from google-workspace, saas-data-apis, document-tools)
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [AIRTABLE_API_KEY, LINEAR_API_KEY, NOTION_API_KEY]
  commands: [python3, curl]
required_credential_files:
  - path: google_token.json
    description: Google OAuth2 token (created by setup script)
  - path: google_client_secret.json
    description: Google OAuth2 client credentials
metadata:
  hermes:
    tags: [Google, Gmail, Calendar, Drive, Sheets, Airtable, Linear, Notion, OCR, PDF, PowerPoint, Documents, Productivity]
    supersedes: [google-workspace, saas-data-apis, document-tools, ocr-and-documents, powerpoint, airtable, linear, notion]
---

# Workspace Tools

Unified skill for workspace productivity: Google Workspace integration, SaaS data management APIs, and document creation/extraction.

**Pick your section:**
- [Section I: Google Workspace](#section-i-google-workspace) — Gmail, Calendar, Drive, Contacts, Sheets, Docs via OAuth
- [Section II: SaaS Data APIs](#section-ii-saas-data-apis) — Airtable, Linear, Notion via REST/GraphQL
- [Section III: Document Tools](#section-iii-document-tools) — OCR, PDF extraction, YouTube transcripts, PowerPoint

---

# Section I: Google Workspace

Gmail, Calendar, Drive, Contacts, Sheets, and Docs through Hermes-managed OAuth and a Python CLI wrapper.

## References

- `references/google-workspace/gmail-search-syntax.md` — Gmail search operators

## Scripts

- `scripts/google-workspace/setup.py` — OAuth2 setup (run once to authorize)
- `scripts/google-workspace/google_api.py` — API wrapper CLI
- `scripts/google-workspace/_hermes_home.py` — HERMES_HOME resolver

## First-Time Setup

```bash
GSETUP="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/workspace-tools/scripts/google-workspace/setup.py"
$GSETUP --check
```

If `AUTHENTICATED`, skip to Usage. Otherwise:

1. **Triage:** Ask if they need email only (use `himalaya` instead), email+calendar, or full workspace
2. **OAuth credentials:** User creates Desktop App OAuth client at Google Cloud Console, downloads JSON
3. **Auth URL:** `$GSETUP --auth-url --services email,calendar --format json`
4. **Exchange code:** `$GSETUP --auth-code "URL_OR_CODE" --format json`
5. **Verify:** `$GSETUP --check`

## Usage

```bash
GAPI="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/workspace-tools/scripts/google-workspace/google_api.py"
```

### Gmail

```bash
$GAPI gmail search "is:unread" --max 10
$GAPI gmail get MESSAGE_ID
$GAPI gmail send --to user@example.com --subject "Hello" --body "Message text"
$GAPI gmail reply MESSAGE_ID --body "Thanks, that works."
$GAPI gmail labels
$GAPI gmail modify MESSAGE_ID --remove-labels UNREAD
```

### Calendar

```bash
$GAPI calendar list
$GAPI calendar create --summary "Team Standup" --start 2026-03-01T10:00:00-06:00 --end 2026-03-01T10:30:00-06:00
$GAPI calendar delete EVENT_ID
```

### Drive

```bash
$GAPI drive search "quarterly report" --max 10
$GAPI drive upload /path/to/report.pdf
$GAPI drive download FILE_ID --output ~/doc.pdf
$GAPI drive create-folder "Reports" --parent FOLDER_ID
$GAPI drive share FILE_ID --email alice@example.com --role reader
$GAPI drive delete FILE_ID
```

### Sheets & Docs

```bash
$GAPI sheets create --title "Q4 Budget"
$GAPI sheets get SHEET_ID "Sheet1!A1:D10"
$GAPI docs get DOC_ID
$GAPI docs create --title "Meeting Notes" --body "First paragraph..."
$GAPI docs append DOC_ID --text "Additional content"
```

## Rules

1. **Never send email, create/delete events, delete files, or share without user confirmation.**
2. **Check auth before first use** — run `setup.py --check`
3. **Calendar times must include timezone** — ISO 8601 with offset or UTC `Z`
4. **Respect rate limits** — batch reads when possible

---

# Section II: SaaS Data APIs

Manage data across Airtable, Linear, and Notion via REST/GraphQL APIs.

## General Principles

- Use `terminal` with `curl` for API calls — not `web_extract` or `browser`
- Parse JSON with `python3 -m json.tool` or `jq`
- Check error arrays — HTTP 200 can still contain errors

## References

- `references/saas-data-apis/linear-api-reference.md` — Detailed Linear queries and mutations
- `references/saas-data-apis/notion-block-types.md` — Full Notion block type reference
- `scripts/saas-data-apis/linear_api.py` — Zero-dependency Linear CLI helper

### Section II-A: Airtable

Spreadsheet-database hybrid. REST API at `https://api.airtable.com/v0/`.

**Setup:** Create personal access token at https://airtable.com/create/tokens, set `AIRTABLE_API_KEY`.

```bash
curl -s "https://api.airtable.com/v0/meta/bases" -H "Authorization: Bearer $AIRTABLE_API_KEY"
curl -s "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}" -H "Authorization: Bearer $AIRTABLE_API_KEY"
curl -s -X POST "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" -H "Content-Type: application/json" \
  -d '{"fields": {"Name": "New Record"}}'
curl -s "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}?filterByFormula={Status}='Active'" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY"
```

Max 100 records/page; use `offset` for pagination. Webhooks supported.

### Section II-B: Linear

Issue tracking via GraphQL at `https://api.linear.app/graphql`.

**Setup:** Get API key from Linear Settings > Account > Security, set `LINEAR_API_KEY`.

```bash
# Python helper
python3 scripts/saas-data-apis/linear_api.py whoami
python3 scripts/saas-data-apis/linear_api.py list-issues
python3 scripts/saas-data-apis/linear_api.py get-issue ENG-42
python3 scripts/saas-data-apis/linear_api.py create-issue --team TEAM_UUID --title "Bug fix" --description "Details"

# Direct GraphQL
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "{ issues(first: 20) { nodes { identifier title state { name } url } } }"}'
```

Auth: `Authorization: $LINEAR_API_KEY` (no Bearer prefix). Rate: 5,000 req/hour. Priority: 0=None, 1=Urgent, 2=High, 3=Medium, 4=Low.

### Section II-C: Notion

Notes, databases, documents. Two paths: `ntn` CLI (preferred) or HTTP + curl.

**Setup:** Create integration at https://notion.so/my-integrations, set `NOTION_API_KEY`, share pages with integration.

```bash
# ntn CLI (preferred)
ntn api v1/search query="page title"
ntn api v1/pages/{page_id}/markdown
ntn api v1/pages parent[page_id]=xxx properties[title][0][text][content]="Notes" markdown="# Agenda"

# HTTP fallback
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer *** \
  -H "Notion-Version: 2025-09-03" -H "Content-Type: application/json" \
  -d '{"query": "page title"}'
```

API Version: `Notion-Version: 2025-09-03`. Properties: title, rich_text, select, multi_select, date, checkbox, number.

---

# Section III: Document Tools

Create, read, edit, and extract content from documents.

## Quick Reference

| Source | Method | Command |
|--------|--------|---------|
| PDF (text) | PyMuPDF | `scripts/document-tools/extract_pymupdf.py file.pdf` |
| PDF (scanned) | Vision model | `vision_analyze(image_url=...)` |
| YouTube | Transcript fetch | `scripts/document-tools/fetch_transcript.py URL` |
| Markdown | Marker | `scripts/document-tools/extract_marker.py file.pdf` |
| Image | Vision model | `vision_analyze(image_url="path")` |

## PDF, OCR, Transcripts

```bash
# PDF text extraction
python3 scripts/document-tools/extract_pymupdf.py document.pdf
python3 scripts/document-tools/extract_pymupdf.py document.pdf --pages 1-5 --extract-images --output-dir ./output

# YouTube transcripts
python3 scripts/document-tools/fetch_transcript.py "https://youtube.com/watch?v=ID" --structured --format srt

# OCR with Marker
python3 scripts/document-tools/extract_marker.py scanned.pdf --quality high

# PPTX cleanup
python3 scripts/document-tools/clean.py deck.pptx --output cleaned.pptx
```

## PowerPoint (.pptx)

```python
from pptx import Presentation
from pptx.util import Inches, Pt
prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[6])
txBox = slide.shapes.add_textbox(Inches(1), Inches(1), Inches(8), Inches(4))
tf = txBox.text_frame
tf.word_wrap = True
p = tf.paragraphs[0]
p.text = "Hello, World!"
p.font.size = Pt(36)
prs.save("output.pptx")
```

## References

| File | Contents |
|------|----------|
| `references/document-tools/youtube-output-formats.md` | Transcript format specs |
| `references/document-tools/pptxgenjs.md` | pptxgenjs library (Node.js) |
| `references/document-tools/editing.md` | Direct OOXML editing techniques |
| `scripts/document-tools/office/` | OOXML schema definitions |