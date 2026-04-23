# Crawler Policy — NewsRadar

**User-Agent:** `NewsRadarCrawler/0.1-alpha`

**Project:** https://github.com/newsradar-404/newsradar
**Contact:** newsradar-ops@pm.me

## What we do

We collect **public metadata** from sitemaps of Brazilian news outlets and enrich it with **OpenGraph meta-tags** from each article page. We apply AI (currently the OpenAI API) over this metadata for editorial framing analysis — **without downloading, storing, republishing, or training on** the full article content.

## What we do NOT do

- ❌ **We don't download full article content.** Only the canonical title (from the sitemap or `og:title`), short summary (`og:description`), featured image (`og:image`), and publication date.
- ❌ **We don't train models** on any collected text. Analysis runs through the API (Chat Completions, stateless) and only the structured result (classification + sentiment + generated reasoning) is persisted.
- ❌ **We don't republish text** from outlets. The dashboard displays only: title, summary (OG), and a link to the original portal.
- ❌ **We don't scrape internal pages**, authenticated APIs, or paywalled areas.
- ❌ **We don't store the HTML** of enriched pages — only the parsed meta-tags.

## Operating principles

### 1. Transparency
- Identifiable User-Agent with a public link to the project policy
- Contact channel available (email)
- This policy is versioned in a public repository

### 2. Technical limits
- **Rate limit:** at most 1 request/second per host, no aggressive parallelism
- **Collection window:** "today" + "yesterday" (BRT) — we don't crawl historical content beyond that
- **Timeout:** 30 seconds per request; if a portal is slow, we back off
- **Cache:** we avoid refetch; URLs already analyzed are not reprocessed

### 3. Honoring `robots.txt`
- **Nominal Disallow to our bot:** we stop that source immediately
- **Disallow to catch-all `*`:** reviewed case-by-case — `Disallow: /private/` does not affect public sitemaps; `Disallow: /` total means we stop
- **Disallow to AI bots nominally (e.g., `ClaudeBot`, `GPTBot`):** we read this as a signal of editorial posture, but strictly respect the intent regarding bots that identify as AI — we don't use disguised UAs. Our UA is its own and does not identify as AI.

### 4. Private output access
- **The radar's output is fully private** — hosted on LAN via Tailscale
- Stakeholders are invited individually (Tailscale ACL control)
- **Never exposed to the public internet** — there is no indexable `.com` URL
- Critical framing: our activity is, in scale and nature, equivalent to **a media analyst doing manual clipping** — only automated. The final product remains internal, not redistributed.

### 5. Reactive good faith
- If an outlet contacts us directly requesting that we stop, we stop that source immediately and respond
- If we discover that our collection is causing problems (load, duplication, misuse of data), we adjust or stop
- We don't dispute formal requests legally if they arrive

## robots.txt vocabulary — our interpretation

Many Brazilian portals today list dozens of AI bots in their `robots.txt`:

```
User-agent: ClaudeBot      Disallow: /
User-agent: GPTBot         Disallow: /
User-agent: anthropic-ai   Disallow: /
... (long list)
User-agent: *              Allow: /
```

**Our interpretation:**
- The block targets **bots that self-identify as AI** for crawling/training/summarization at large scale
- Our bot is **none of those** — it is named, with a specific purpose, and limited operation
- The `User-agent: *` Allow technically covers us
- **But the spirit of robots.txt** is to avoid AI use over content — and we ARE using AI (OpenAI) over metadata that the sitemap itself exposes publicly for bot consumption

**How we reconcile this:**
- We collect only what the sitemap **deliberately exposes for bots** (that's the purpose of a sitemap)
- We use AI **only on metadata** (title, short summary) that is already visible in any social link preview
- **We don't reprocess full content** — we never use the article body itself
- Output is **private** — we don't feed a public index, we don't generate alternative summaries for mass consumption

## Status per source (April 2026)

| Outlet | Explicit UA in robots? | Our posture |
|---|---|---|
| G1 | to verify | operating — review robots |
| UOL | blocks nominal AI bots (ClaudeBot, GPTBot, etc.); `*` Allow | operating — our UA passes |
| Metrópoles | to verify | operating |
| Folha de S.Paulo | blocks nominal AI bots; `Googlebot-News` Allow; `*` not blocked | operating — our UA passes |
| Gazeta do Povo | to verify | pending |

## Policy review

This policy is reviewed at every new source addition and at every milestone. Changes are committed with justification.

**Last revised:** 2026-04-23
