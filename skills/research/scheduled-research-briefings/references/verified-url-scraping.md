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
| The Register | `theregister.com` | **SSR — curl-friendly**. Good for UK data breach/security news. Brave or Google News RSS |

## Google News RSS (Fallback Search)

When Brave search and web_search tools are both unavailable or returning empty results, **Google News RSS** is the most reliable fallback. It returns structured XML with real titles, URLs, publication dates, and source names.

```bash
# General search
curl -s "https://news.google.com/rss/search?q=QUERY+HERE" | \
  python3 -c "
import sys, re, xml.etree.ElementTree as ET
feed = ET.fromstring(sys.stdin.read())
ns = {'a': 'http://www.w3.org/2005/Atom'}
for item in feed.findall('.//a:item', ns)[:20]:
    title = item.find('a:title', ns).text
    link = item.find('a:link', ns).attrib.get('href', '')
    # Google links use proxy — extract real URL
    real = re.search(r'url=(https?://[^&]+)', link)
    if real: link = real.group(1)
    pub = item.find('a:published', ns).text[:10]
    source = item.find('a:source', ns).text if item.find('a:source', ns) is not None else '?'
    print(f'{pub} | {source} | {title}')
    print(f'  {link}')
"
```

### Date Filtering

Google News RSS supports `after:` and `before:` date operators for time-bounded research:

```bash
curl -s "https://news.google.com/rss/search?q=data+breach+ICO+after:2025-05-01+before:2025-06-01" | ...
```

### Site-Specific RSS

Combine with `site:` operator:

```bash
curl -s "https://news.google.com/rss/search?q=site:theregister.com+data+breach+2025" | ...
```

**Limitations:** ~20 results per query. May need multiple queries for comprehensive coverage. Google proxy URLs need stripping (the regex above handles this).

## The Register (theregister.com)

Server-side rendered — fully curl-scrapeable. Rich source for UK tech/data protection/security news.

```bash
# Search via Google News RSS (most reliable)
curl -s "https://news.google.com/rss/search?q=site:theregister.com+data+breach+after:2025-05-01+before:2025-06-01" | ...

# Or Brave search + grep
curl -s "https://search.brave.com/search?q=site:theregister.com+data+breach+2025" \
  -H "User-Agent: Mozilla/5.0" | \
  grep -oP 'href="(https://www\.theregister\.com/[0-99/]+[^"]*)"'
```

## Curl + Python3 Parsing Pattern

For extracting structured data from server-side rendered HTML when grep is insufficient:

```bash
curl -sL "URL" -H "User-Agent: Mozilla/5.0" | python3 -c "
import sys, re
html = sys.stdin.read()
# Example: extract article titles and dates from a news listing page
items = re.findall(r'<h2[^>]*>(.*?)</h2>', html, re.DOTALL)
dates = re.findall(r'class=\"date\">(.*?)<', html)
for i, item in enumerate(items[:20]):
    title = re.sub(r'<[^>]+>', '', item).strip()
    print(f'{dates[i] if i < len(dates) else \"?\"} | {title}')
"
```

This pattern is more flexible than grep for multi-line matches, attribute extraction, and when you need to strip HTML tags.

## Anti-Hallucination Checklist for Cron Prompts

Include these rules in any research briefing cron prompt:

1. "NEVER fabricate URLs. Only include URLs extracted from actual web pages or search results."
2. "Verify each link exists before including it in the briefing."
3. "If a search returns no results, say 'No new updates' — do not invent articles."
4. Provide explicit curl commands in the prompt so the agent follows a proven path rather than improvising.
