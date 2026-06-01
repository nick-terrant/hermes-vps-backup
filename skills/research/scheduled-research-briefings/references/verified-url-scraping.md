# Verified URL Scraping for Research Briefings

When building research briefings, LLMs may hallucinate plausible-looking URLs. Use these terminal-based techniques to get **real, verified** links.

## Search Engine Scraping (Brave)

Brave search returns consistent, extractable results via curl:

```bash
# General search
curl -s "https://search.brave.com/search?q=QUERY+HERE" \
  -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36" | \
  grep -oP 'href="(https://[^"]+)"' | grep -v 'brave\|favicon'

# Site-specific search
curl -s "https://search.brave.com/search?q=site:ico.org.uk/news+2026&source=web" \
  -H "User-Agent: Mozilla/5.0" | \
  grep -oP 'href="https://ico\.org\.uk/about-the-ico/media-centre/news-and-blogs/[0-9]+[^"]*"'
```

**Note:** DuckDuckGo HTML search (`html.duckduckgo.com/html/?q=...`) is unreliable — often returns 0 results for site-specific queries. Prefer Brave.

## Title Verification

Always verify the actual title of a URL before including it:

```bash
curl -sL "URL" -H "User-Agent: Mozilla/5.0" | grep -oP '<title>\K[^<]+'
```

## IAPP-Specific: Algolia JSON Extraction

IAPP embeds article data in Algolia JSON in the page source. Scrape it directly:

```bash
curl -sL "https://iapp.org/news/" -H "User-Agent: Mozilla/5.0" | python3 -c "
import sys, re
html = sys.stdin.read()
hits = re.findall(r'\"headline\":\"([^\"]+)\",\"date\":\"([^\"]+)\"', html)
urls = re.findall(r'\"url\":\"(/news/a/[^\"]+)\"', html)
for i, (headline, date) in enumerate(hits[:20]):
    hl = headline.lower()
    if any(k in hl for k in ['uk','ico','ai ','artificial','governance','gdpr','europe']):
        print(f'{date} | {headline}')
        print(f'  https://iapp.org{urls[i]}')
        print()
"
```

## Key UK Data Protection Sources

| Source | Base URL | Scrape Method |
|--------|----------|---------------|
| ICO News | `ico.org.uk/about-the-ico/media-centre/news-and-blogs/` | Brave search + title verification |
| IAPP | `iapp.org/news/` | Algolia JSON extraction (see above) |
| Bristows Inquisitive Minds | `inquisitiveminds.bristows.com` | Brave search |
| Pinsent Masons Out-Law | `out-law.com/tag/data-protection/` | JS-rendered, use Brave search |
| Fieldfisher | `fieldfisher.com/en/insights` | JS-rendered, use Brave search |
| Clyde & Co | `clydeco.com/en/insights` | JS-rendered, use Brave search |

## Anti-Hallucination Checklist for Cron Prompts

Include these rules in any research briefing cron prompt:

1. "NEVER fabricate URLs. Only include URLs extracted from actual web pages or search results."
2. "Verify each link exists before including it in the briefing."
3. "If a search returns no results, say 'No new updates' — do not invent articles."
4. Provide explicit curl commands in the prompt so the agent follows a proven path rather than improvising.
