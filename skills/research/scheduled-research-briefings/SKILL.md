---
name: scheduled-research-briefings
description: "Set up automated cron-based research briefings — daily news digests, domain monitoring, source aggregation with headline-level summaries."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [cron, research, briefings, news, monitoring, scheduled]
    related_skills: [blogwatcher]
---

# Scheduled Research Briefings

Set up automated, recurring research briefings delivered to the user's messaging platform via Hermes cron jobs. Covers daily news digests, domain-specific monitoring, multi-source aggregation, and headline-level summaries.

Use this skill when the user asks for:
- A daily/weekly automated news digest or briefing
- Scheduled monitoring of specific sources, topics, or domains
- Recurring research summaries delivered at a set time

This is different from `blogwatcher` (RSS/Atom feed tracking via blogwatcher-cli). This skill covers **web-search-based research briefings** powered by cron jobs.

## Workflow

### 1. Gather Requirements

Ask the user (or infer from context):
- **Topic/domain** — what to cover
- **Sources** — specific sites, publications, blogs, or let the agent pick
- **Frequency** — daily, weekly, on specific weekdays
- **Time** — delivery time (always convert to UTC for cron; check server timezone first)
- **Depth** — headlines only, one-paragraph summaries, or deep-dive
- **Delivery** — which chat/channel (default: origin/current chat)

### 2. Create the Cron Job

Use the `cronjob` tool with:
- `schedule`: cron expression or natural time (e.g. `"0 6 * * *"` for 6 AM UTC)
- `deliver`: `"origin"` (sends to current chat)
- `enabled_toolsets`: `["web"]` (research only needs web search/extract)
- `prompt`: a self-contained research prompt (see Prompt Design below)

### 3. Test with a Live Run

Before relying on the schedule, do a live test:
- Option A: `cronjob(action="run", job_id=...)` — triggers the cron job directly
- Option B: `hermes chat -q '<research prompt>' --toolsets web -Q` via terminal — more reliable fallback, works even if cron hits iteration limits

If the cron run produces incomplete output, use the terminal fallback to demonstrate the format, then refine the cron prompt.

## Prompt Design

### Efficiency is Critical

Cron jobs have a **3-minute hard interrupt** and limited iteration budget. A prompt that asks for 10+ sequential web searches will exhaust the budget before completing. Design prompts to be efficient:

**Bad:** List 7 specific law firm URLs and ask the agent to visit each one, then also search IAPP, then also check ICO news, then also search general news.

**Good:** Give 3-4 broad web search queries that cover all sources at once via `site:` operators and OR logic. Let the agent consolidate.

### Template Prompt

```
You are producing a [daily/weekly] [headline/summary] briefing on [TOPIC] for a [USER CONTEXT].

## Sources to Search

Run web searches using these queries (run all, then compile):
1. "site:primary-source.com keyword1 OR keyword2"
2. "site:secondary-source.com keyword1 OR keyword2"
3. "general query for [topic] recent news [date range]"

## Output Format

[CATEGORY HEADER] (e.g. 📰 News, 🤖 AI Updates, ⚖️ Legal)

For each item:
- **Source** — publication or site name
- **Headline** — one line
- **Summary** — one sentence, why it matters
- **Link** — direct URL

If no new items in a category, say "No new updates."
Keep the whole briefing under [N] lines.

[SILENT instruction: If genuinely nothing new to report, respond with exactly "[SILENT]" to suppress delivery.]
```

## Pitfalls

### Cron Iteration Budget

**Problem:** Cron prompts that require many sequential tool calls (search, extract, search, extract, ...) may exhaust the iteration budget before completing. The agent generates planned searches in its response text but doesn't get to execute them all.

**Sign:** Cron output at `~/.hermes/cron/output/<job_id>/` contains only the prompt + a single "Let me check..." line with no actual results.

**Fix:** Consolidate queries. Use fewer, broader searches. Use `site:` operators to cover multiple sources in one query. If you need to check 5+ sources, consider chaining two cron jobs (`context_from`) or using a pre-run script.

### delegate_task Not Suitable for Multi-Step Web Research

**Problem:** `delegate_task` with certain models (notably smaller/faster models) may exhaust iteration budget planning tool calls without executing them. The subagent's summary will say "I'll run searches now" but contain zero results.

**Fix:** Do the research directly in the main session using web tools, or use `hermes chat -q 'prompt' --toolsets web -Q` via terminal, which runs a full agent loop with its own iteration budget.

### Timezone Mismatch

**Problem:** Server runs on UTC. User's local time may differ (e.g. UK is UTC+0 in winter, UTC+1 in summer/BST).

**Fix:** Check server timezone first (`timedatectl` or `date +%Z`), then offset the cron schedule accordingly. Remind the user of the actual UTC time being set.

### Delivery Visibility

**Problem:** Cron output delivered to "origin" appears as a separate message in the chat, not nested in the current conversation thread. User may not see it if they're focused on the active conversation.

**Fix:** Explain this to the user. Cron output is also stored at `~/.hermes/cron/output/<job_id>/<timestamp>.md` for reference. For the initial test/demo, prefer running the briefing live in-conversation so the user sees the format immediately.

## Output Storage

Cron job output is persisted at:
```
~/.hermes/cron/output/<job_id>/<timestamp>.md
```

Use this path to inspect past runs when debugging or reviewing.

## Example: Daily UK Data Protection Briefing

```python
cronjob(
    action="create",
    name="UK Data Protection & AI News Briefing",
    schedule="0 6 * * *",  # 6 AM UTC = 7 AM BST
    deliver="origin",
    enabled_toolsets=["web"],
    prompt="""
    Daily headline briefing on UK data protection and AI news.

    Search queries to run:
    1. "UK data protection news [current_month] [current_year]"
    2. "site:iapp.org UK OR ICO OR artificial intelligence"
    3. "site:out-law.com data protection 2026"

    Format: Headlines grouped by category (UK DP News, AI & Regulation, Lawyer Blogs).
    Each item: Source | Headline | One sentence | Link.
    Under 40 lines. Headlines only — user drills down on interest.
    """
)
```
