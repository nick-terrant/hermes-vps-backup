---
name: saas-data-apis
description: "SaaS data management: Airtable, Linear, and Notion via REST API and CLI."
version: 1.0.0
author: Hermes Agent (consolidated from airtable, linear, notion)
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [AIRTABLE_API_KEY, LINEAR_API_KEY, NOTION_API_KEY]
  commands: [curl]
metadata:
  hermes:
    tags: [Airtable, Linear, Notion, Productivity, API, Database, Issues, CRM]
    supersedes: [airtable, linear, notion]
---

# SaaS Data Management APIs

Manage data across Airtable, Linear, and Notion via their REST APIs and CLIs. All three services follow the same pattern: authenticate with an API key, make HTTP requests, parse JSON responses.

**Pick the section for your service:**
- [Section I: Airtable](#section-i-airtable) — Spreadsheet-database hybrid for structured data
- [Section II: Linear](#section-ii-linear) — Issue tracking and project management via GraphQL
- [Section III: Notion](#section-iii-notion) — Notes, databases, documents, and Workers via `ntn` CLI or HTTP

## General Principles

- Always use `terminal` tool with `curl` for API calls — do NOT use `web_extract` or `browser`
- Parse JSON output with `python3 -m json.tool` or `jq`
- Check error arrays in responses — HTTP 200 can still contain errors
- Rate limits vary by service; check headers (`X-RateLimit-*`) when available

---

# Section I: Airtable

Spreadsheet-database hybrid. REST API at `https://api.airtable.com/v0/`.

## Setup

1. Create a personal access token at https://airtable.com/create/tokens
2. Set `AIRTABLE_API_KEY` in your environment (via `hermes setup` or env config)

## API Pattern

```bash
# List bases
curl -s "https://api.airtable.com/v0/meta/bases" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool

# List records from a table
curl -s "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool

# Create a record
curl -s -X POST "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fields": {"Name": "New Record", "Status": "Active"}}' | python3 -m json.tool

# Update a record
curl -s -X PATCH "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fields": {"Status": "Done"}}' | python3 -m json.tool

# Delete a record
curl -s -X DELETE "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY"
```

## Common Operations

| Task | Endpoint | Method |
|------|----------|--------|
| List bases | `/meta/bases` | GET |
| List tables in base | `/meta/bases/{BASE_ID}/tables` | GET |
| List records | `/{BASE_ID}/{TABLE_NAME}` | GET |
| Get record | `/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}` | GET |
| Create record | `/{BASE_ID}/{TABLE_NAME}` | POST |
| Update record | `/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}` | PATCH |
| Delete record | `/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}` | DELETE |

## Filtering & Sorting

```bash
# Filter by field value
curl -s "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}?filterByFormula={Status}='Active'" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY"

# Sort and limit
curl -s "https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}?sort[0][field]=CreatedTime&sort[0][direction]=desc&pageSize=100" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY"
```

## Webhooks

Airtable supports webhooks for change notifications:
```bash
# Create webhook
curl -s -X POST "https://api.airtable.com/v0/{BASE_ID}/webhooks" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"notificationUrl": "https://your-server.com/webhook"}'
```

## Notes

- Auth header is always `Authorization: Bearer $AIRTABLE_API_KEY`
- `TABLE_NAME` must be URL-encoded if it contains spaces
- Max 100 records per page; use `offset` for pagination
- Attachments require multi-part form upload

---

# Section II: Linear

Issue tracking and project management via GraphQL API.

## Setup

1. Get a personal API key from **Linear Settings > Account > Security & access > Personal API keys** (https://linear.app/settings/account/security)
2. Set `LINEAR_API_KEY` in your environment

## API Basics

- **Endpoint:** `https://api.linear.app/graphql` (POST only)
- **Auth header:** `Authorization: $LINEAR_API_KEY` (no "Bearer" prefix for API keys)
- Both UUIDs and short identifiers (`ENG-123`) work for `issue(id:)`

### Python Helper Script

This skill ships a zero-dependency CLI at `scripts/linear_api.py`:

```bash
python3 scripts/linear_api.py whoami
python3 scripts/linear_api.py list-teams
python3 scripts/linear_api.py get-issue ENG-42
python3 scripts/linear_api.py list-issues
python3 scripts/linear_api.py create-issue --team TEAM_UUID --title "Bug fix" --description "Details"
python3 scripts/linear_api.py search-issues "login bug"
```

All subcommands: `whoami`, `list-teams`, `list-projects`, `list-states`, `list-issues`, `get-issue`, `search-issues`, `create-issue`, `update-issue`, `update-status`, `add-comment`, `list-documents`, `get-document`, `search-documents`, `raw`.

### Common Queries

```bash
# Get current user
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ viewer { id name email } }"}'

# List issues (first 20)
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ issues(first: 20) { nodes { identifier title priority state { name type } assignee { name } team { key } url } } }"}'

# My assigned issues
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ viewer { assignedIssues(first: 25) { nodes { identifier title state { name } priority url } } } }"}'

# Get single issue by identifier
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ issue(id: \"ENG-123\") { id identifier title description priority state { id name type } assignee { name } comments { nodes { body } } url } }"}'

# Search issues
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ issueSearch(query: \"bug login\", first: 10) { nodes { identifier title state { name } url } } }"}'
```

### Common Mutations

```bash
# Create issue
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "mutation($input: IssueCreateInput!) { issueCreate(input: $input) { success issue { id identifier title url } } }", "variables": {"input": {"teamId": "TEAM_UUID", "title": "Fix login bug", "priority": 2}}}'

# Update issue status (need state UUID from workflow states query)
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "mutation { issueUpdate(id: \"ENG-123\", input: { stateId: \"STATE_UUID\" }) { success issue { identifier state { name } } } }"}'

# Add comment
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "mutation { commentCreate(input: { issueId: \"ISSUE_UUID\", body: \"Investigated. Root cause is X.\" }) { success comment { id } } }"}'
```

### Workflow States

| Type | Description |
|------|-------------|
| `triage` | Incoming issues |
| `backlog` | Not yet planned |
| `unstarted` | Ready but not started |
| `started` | In progress |
| `completed` | Done |
| `canceled` | Won't do |

Priority: 0=None, 1=Urgent, 2=High, 3=Medium, 4=Low.

### Rate Limits

5,000 requests/hour. 3,000,000 complexity points/hour. Always use `first: N` to limit results.

## Notes

- Always check `errors` array in GraphQL responses
- Default page size: 50, max: 250
- If `stateId` omitted, Linear defaults to first backlog state

---

# Section III: Notion

Notes, databases, documents, and Workers. Two access paths: `ntn` CLI (preferred, macOS/Linux) or HTTP + curl (cross-platform).

## Setup

### 1. Get integration token
1. Create integration at https://notion.so/my-integrations
2. Copy API key (starts with `ntn_` or `secret_`)
3. Set `NOTION_API_KEY` in `~/.hermes/.env`
4. **Share target pages/databases with the integration** in Notion (page menu → Connect to → your integration)

### 2. Install `ntn` CLI (preferred, macOS/Linux)
```bash
curl -fsSL https://ntn.dev | bash
# Or: npm install --global ntn (needs Node 22+)
```

Set env vars for headless auth:
```bash
export NOTION_API_TOKEN=$NOTION_API_KEY
export NOTION_KEYRING=0
```

### 3. API Version
`Notion-Version: 2025-09-03` required on all HTTP requests. In this version, "databases" are called "data sources" in the API.

## Path A: `ntn` CLI

```bash
# Search
ntn api v1/search query="page title"

# Read page as Markdown (agent-friendly)
ntn api v1/pages/{page_id}/markdown

# Create page from Markdown
ntn api v1/pages \
  parent[page_id]=xxx \
  properties[title][0][text][content]="Notes" \
  markdown="# Agenda\n- Q3 roadmap"

# Patch page with Markdown
ntn api v1/pages/{page_id}/markdown -X PATCH \
  markdown="## Update\nShipped the prototype."

# Query database (data source)
ntn api v1/data_sources/{data_source_id}/query -X POST \
  filter[property]=Status filter[select][equals]=Active

# File uploads (one-liner — biggest CLI win)
ntn files create < photo.png
```

## Path B: HTTP + curl

```bash
# Search
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"query": "page title"}'

# Read page as Markdown
curl -s "https://api.notion.com/v1/pages/{page_id}/markdown" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"

# Create page in database
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"parent": {"database_id": "xxx"}, "properties": {"Name": {"title": [{"text": {"content": "New Item"}}]}, "Status": {"select": {"name": "Todo"}}}}'

# Append blocks
curl -s -X PATCH "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"children": [{"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Hello from Hermes!"}}]}}]}'
```

## Property Types

- Title: `{"title": [{"text": {"content": "..."}}]}`
- Rich text: `{"rich_text": [{"text": {"content": "..."}}]}`
- Select: `{"select": {"name": "Option"}}`
- Multi-select: `{"multi_select": [{"name": "A"}]}`
- Date: `{"date": {"start": "2026-01-15"}}`
- Checkbox: `{"checkbox": true}`
- Number: `{"number": 42}`

## API Version 2025-09-03 — Databases vs Data Sources

- Use `/data_sources/` endpoints for queries and retrieval
- `database_id` when creating pages: `parent: {"database_id": "..."}`
- `data_source_id` when querying: `POST /v1/data_sources/{id}/query`

## Notion Workers (advanced, requires `ntn`)

TypeScript programs hosted by Notion for syncs, tools, and webhooks. Deploying requires Business or Enterprise plan.

```bash
ntn workers new my-worker
cd my-worker
ntn workers deploy --name my-worker
```

## References

- **[notion-block-types.md](references/notion-block-types.md)** — Full block type reference
- **[linear-api-reference.md](references/linear-api-reference.md)** — Detailed Linear API queries and mutations reference
