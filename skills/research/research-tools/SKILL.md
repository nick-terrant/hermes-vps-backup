---
name: research-tools
description: "Unified research toolkit: arXiv paper search with Semantic Scholar citations, Karpathy's LLM Wiki knowledge base management, and blog/RSS feed monitoring via blogwatcher."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [research, arxiv, papers, blog, rss, wiki, monitoring]
    category: research
---

# Research Tools

A consolidated skill covering three research domains: academic paper discovery, persistent knowledge-base management, and blog/RSS feed monitoring.

## ArXiv Paper Search

Search and retrieve academic papers from arXiv via their free REST API, with supplementary citation and recommendation data from Semantic Scholar. No API key required — just curl.

**Capabilities:**
- Search papers by keyword, author, category, or arXiv ID
- Parse Atom XML results into clean, readable output
- Fetch citations, references, and related papers via Semantic Scholar
- Generate BibTeX entries
- Complete research workflow: discover → assess impact → read → find related

**Quick commands:**
| Action | Command |
|--------|---------|
| Search papers | `curl "https://export.arxiv.org/api/query?search_query=all:QUERY&max_results=5"` |
| Get specific paper | `curl "https://export.arxiv.org/api/query?id_list=2402.03300"` |
| Citation data | `curl -s "https://api.semanticscholar.org/graph/v1/paper/arXiv:ID?fields=citationCount,influentialCitationCount"` |

See [references/arxiv.md](references/arxiv.md) for the full guide including search syntax, Boolean operators, sorting, pagination, and the helper script.

## LLM Wiki

Build and maintain a persistent, compounding knowledge base as interlinked markdown files. Based on Andrej Karpathy's LLM Wiki pattern. Unlike traditional RAG (which rediscovers knowledge per query), the wiki compiles knowledge once and keeps it current with cross-references.

**Capabilities:**
- Three-layer architecture: raw sources (immutable) → wiki pages (agent-managed) → schema (conventions)
- Ingest, query, and lint operations
- Cross-referencing with `[[wikilinks]]`
- Obsidian integration (works as a vault out of the box)
- Automated health checks: orphans, broken links, stale content, contradictions

**Key operations:**
| Operation | Description |
|-----------|-------------|
| Ingest | Capture sources, create/update wiki pages, cross-reference |
| Query | Search index + pages, synthesize answers, file valuable results |
| Lint | Orphan pages, broken wikilinks, stale content, contradictions |

**Location:** Set via `WIKI_PATH` env var (defaults to `~/wiki`).

See [references/llm-wiki.md](references/llm-wiki.md) for the full guide including initialization, SCHEMA.md template, bulk ingest, and Obsidian headless sync.

## Blog/RSS Monitoring

Track blog and RSS/Atom feed updates with the `blogwatcher-cli` tool. Supports automatic feed discovery, HTML scraping fallback, OPML import, and read/unread article management.

**Capabilities:**
- Add blogs by URL (auto-discovers RSS/Atom feeds)
- HTML scraping fallback for sites without feeds
- OPML import from Feedly, Inoreader, NewsBlur, etc.
- Scan for new articles, manage read/unread state
- Category-based filtering

**Quick commands:**
| Action | Command |
|--------|---------|
| Add blog | `blogwatcher-cli add "My Blog" https://example.com` |
| Scan all | `blogwatcher-cli scan` |
| List unread | `blogwatcher-cli articles` |
| Import OPML | `blogwatcher-cli import subscriptions.opml` |

See [references/blogwatcher.md](references/blogwatcher.md) for the full guide including installation methods, Docker usage, and environment variables.

## When to Use

| Scenario | Use This |
|----------|----------|
| Find academic papers on a topic | ArXiv Paper Search |
| Look up a specific paper by ID | ArXiv Paper Search |
| Get citation count or related papers | ArXiv Paper Search (Semantic Scholar) |
| Build a persistent knowledge base | LLM Wiki |
| Ingest a source into an existing wiki | LLM Wiki |
| Query or lint a wiki | LLM Wiki |
| Track blog/RSS feed updates | Blog/RSS Monitoring |
| Import subscriptions from another reader | Blog/RSS Monitoring |
| Generate BibTeX for a citation | ArXiv Paper Search |
| Check wiki health (broken links, orphans) | LLM Wiki (Lint) |

## Notes

- All three sub-tools are free and require minimal dependencies
- ArXiv rate limit: ~1 req/3s; Semantic Scholar: 1 req/s (100/s with API key)
- The LLM Wiki integrates with Obsidian for GUI browsing
- blogwatcher-cli requires installation (Go, Docker, or binary)
