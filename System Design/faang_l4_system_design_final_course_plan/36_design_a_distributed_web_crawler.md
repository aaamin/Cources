# Session 36 — Design a Distributed Web Crawler

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice distributed scheduling, politeness, deduplication, crash recovery, retries, and work prioritization.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- URL frontier and prioritization.
- Canonicalization and URL/content deduplication.
- Robots rules and per-host politeness/rate limits.
- Workers and fetch pipeline.
- Host-based scheduling to avoid hammering one site.
- Retry/backoff and poison pages.
- Visited-state/content storage.
- Recrawl scheduling/freshness.
- Crash recovery and at-least-once work.
- Priority between news freshness and fairness.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Draw frontier → scheduler → workers → parser/dedup → storage → discovered URLs.
- [ ] Change request: prioritize news freshness without overwhelming small sites.
- [ ] Explain worker crash after fetch but before acknowledging the URL.

## **Must Remember for the Interview**

- **Crawler scalability is constrained by politeness/fair scheduling as well as raw worker count.**
- **Deduplicate URLs before repeatedly fetching; content dedup can be a separate stage.**
- **Partition/schedule by host/domain to enforce per-host limits.**
- **At-least-once work means duplicate fetches are possible; make downstream processing tolerant.**
- **Recrawl priority is a freshness/resource trade-off.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Frontier stores pending URLs with priority/schedule.**
- **Scheduler enforces robots/politeness and assigns fetch work.**
- **Canonicalize + deduplicate discovered URLs.**
- **Persist enough frontier state for crash recovery.**
- **Balance freshness against fairness and external-site load.**

## Self-Test Before Marking This Session Complete

- [ ] Did I include politeness/robots?
- [ ] Did I separate URL and content dedup?
- [ ] Did I handle worker crash/retry?
- [ ] Did I define frontier priority?
- [ ] Did I discuss recrawl/freshness?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Prioritize news freshness without overwhelming small sites.

**Score using the 40-point rubric.**


---

**Progress:** Session 36/46  
**Next:** Session 37
