# Crawler Policy — NewsRadar

**User-Agent:** `NewsRadarCrawler/0.1-alpha`

**Project:** https://github.com/newsradar-404/newsradar
**Contact:** newsradar-ops@pm.me

## What we do

We collect **public metadata** from sitemaps of news outlets and enrich it with **OpenGraph meta-tags** from each article page. We apply AI (large language models, local or hosted) over this metadata for editorial framing analysis — **without republishing or training on** the full article content. A narrow exception applies to signed opinion columns; see the dedicated section below.

## What we do NOT do

- ❌ **We don't download full article content for news pieces.** Only the canonical title (from the sitemap or `og:title`), short summary (`og:description`), featured image (`og:image`), and publication date. (For signed opinion columns, see the next section — body extraction is enabled under tight scope with body never published or redistributed.)
- ❌ **We don't train models** on any collected text. LLM calls are stateless; we opt out of provider-level training where available, and only the structured result (classification + sentiment + generated reasoning) is persisted.
- ❌ **We don't republish text** from outlets. The dashboard displays only: title, summary (OG), and a link to the original portal.
- ❌ **We don't scrape internal pages**, authenticated APIs, or paywalled areas.
- ❌ **We don't store the HTML** of enriched pages — only the parsed meta-tags.

## What we DO for signed opinion columns (since Apr 2026)

Tonal analysis of signed opinion columns requires sentence-level text — the headline and OG summary aren't enough to detect sentiment. We extended the pipeline accordingly, with tight scope:

- **Scope:** only articles classified as `opinion` AND authored by a named, public-facing columnist. News articles, editorial-board pieces, and unsigned content are NOT subject to body extraction.

- **How we extract:** Mozilla's Readability algorithm, running locally.

- **What we do with it:** pass the extracted body to a chosen LLM, local or currently available in the market. Only the structured output is persisted in our analytical DB.

- **Where the article body lives:** extracted text is written to a local file that never reaches any public repository. Its only purposes are (a) auditability of the analysis and (b) content-hash comparison to detect stealth edits, so our internal UI can flag "text edited since last analysis" to the operator.

- **What we do NOT do with the body:**
  - We NEVER publish, redistribute, or display it to any audience
  - We do NOT train models on it — LLM calls are stateless; we opt out of provider-level training where the provider offers that control
  - We do NOT expose it via any API or URL
  - We do NOT generate summaries/rewritings that could displace the original

- **Rationale:** tonal analysis of opinion columns is meaningfully different from factual reporting — it captures the rhetorical register of a signed author taking public positions. This is work that media analysts already do manually — reading columns and taking notes on tone. The structured output is OUR analytical product, not the outlet's content. The body itself remains the outlet's property — we handle it exactly as a human analyst would: read, take notes, keep source material for reference, never republish.

- **Scale:** currently we read under 100 signed opinion columns per day across the monitored outlets. Rate limit (1 req/s per host) applies as for sitemap/OG fetches.

- **Opt-out path:** if an outlet or columnist contacts us requesting that their content be excluded from body extraction, we stop for that source / author immediately (see §5 — Reactive good faith).

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
- **The radar's output is fully private** — hosted on a private network, not exposed to the public internet
- Stakeholders are invited individually with per-user access control
- **Never exposed to the public internet** — there is no indexable URL
- Critical framing: our activity is, in scale and nature, equivalent to **a media analyst doing manual clipping** — only automated. The final product remains internal, not redistributed.

### 5. Reactive good faith
- If an outlet contacts us directly requesting that we stop, we stop that source immediately and respond
- If we discover that our collection is causing problems (load, duplication, misuse of data), we adjust or stop
- We don't dispute formal requests legally if they arrive
- **Responsible disclosure:** if anyone finds a radar URL publicly indexed, reachable outside our private network, or otherwise leaked, please notify us immediately at the contact above — we treat this as a critical issue and address it on the same day

## robots.txt vocabulary — our interpretation

Many news portals today list dozens of AI bots in their `robots.txt`:

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
- **But the spirit of robots.txt** is to avoid AI use over content — and we ARE using AI over metadata that the sitemap itself exposes publicly for bot consumption

**How we reconcile this:**
- We collect only what the sitemap **deliberately exposes for bots** (that's the purpose of a sitemap)
- We use AI **only on metadata** (title, short summary) that is already visible in any social link preview
- **We don't reprocess full content** in general, with a single scoped exception covering signed opinion columns (see the dedicated section below for the tight limits applied there)
- Output is **private** — we don't feed a public index, we don't generate alternative summaries for mass consumption

## Status per source (April 2026)

> Note: outlet `robots.txt` policies change frequently. The data below reflects our last review; verify directly at `<outlet>/robots.txt` for the current state.

### United States (reference — not in active crawl scope)

| Outlet | Explicit UA in robots? | Our posture |
|---|---|---|
| The New York Times | blocks `GPTBot`, `ClaudeBot`, `anthropic-ai`, `Google-Extended`, `CCBot`; tight `*` rules | reviewed — would not crawl without explicit permission |
| The Washington Post | blocks `GPTBot`, `ClaudeBot`, `anthropic-ai`; `*` allowed for most paths | reviewed — not in scope |
| CNN | blocks `GPTBot`, `CCBot`; `Googlebot` allowed | reviewed — not in scope |
| AP News | licensing agreement with OpenAI (Jul 2023); `GPTBot` allowed; other AI bots blocked | reviewed — not in scope |
| NPR | blocks `GPTBot`, `ClaudeBot` | reviewed — not in scope |

### Brazil (active crawl scope — representative outlets)

> Note: the table below lists a representative sample of the outlets we actively crawl — including but not limited to these.

| Outlet | Explicit UA in robots? | Our posture |
|---|---|---|
| G1 | to verify | operating — review robots |
| UOL | blocks nominal AI bots (`ClaudeBot`, `GPTBot`, etc.); `*` Allow | operating — our UA passes |
| Metrópoles | to verify | operating |
| Folha de S.Paulo | blocks nominal AI bots; `Googlebot-News` Allow; `*` not blocked | operating — our UA passes |
| Gazeta do Povo | to verify | pending |

## Policy review

This policy is reviewed at every new source addition and at every milestone. Changes are committed with justification.

**Last revised:** 2026-04-24
