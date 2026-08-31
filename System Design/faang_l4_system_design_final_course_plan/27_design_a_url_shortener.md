# Session 27 — Design a URL Shortener

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Use a simple read-heavy service to practice the full interview framework, data model, ID generation, caching, and correctness around aliases.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: create short URL, redirect, optional expiration/custom alias; decide whether analytics is in scope.
- Scale: write volume versus redirect/read volume; latency target; storage retention.
- API/data: `POST /links`, `GET /{code}`; mapping `code → long_url`, owner, created_at, expires_at.
- ID/code generation: random code, encoded sequence, collision handling; understand trade-offs.
- Storage: key lookup workload; SQL or key-value can both be justified.
- Cache: popular redirects, TTL, stale/expired link behavior.
- Correctness: alias uniqueness and custom alias race.
- Abuse/security: malicious URLs/rate limits at recognition depth.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Do a 45-minute first attempt before reading a solution.
- [ ] Change request: support custom aliases and define whether deleted aliases can be reused.
- [ ] Redo only the ID-generation + alias-creation portion after review.

## **Must Remember for the Interview**

- **The redirect path is read-heavy and latency-sensitive; alias creation is correctness-sensitive.**
- **Custom alias uniqueness needs an atomic uniqueness mechanism, not 'check then insert'.**
- **ID generation and storage are separate decisions.**
- **Cache popular mappings, but the database remains the source of truth.**
- **Separate redirect serving from optional analytics if analytics would add synchronous latency.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Flow: clarify → estimate read/write → API/data → code generation → DB → cache → redirect.**
- **Alias creation requires uniqueness; redirects can usually tolerate cache-based reads.**
- **Discuss expiration and cache invalidation.**
- **At scale, hotspots are popular links; CDN/cache may absorb reads.**
- **Be able to compare random-code collision handling versus sequence/Snowflake-style IDs.**

## Self-Test Before Marking This Session Complete

- [ ] Did I state read/write ratio and latency needs?
- [ ] Did I define alias uniqueness behavior?
- [ ] Did I show source of truth and cache policy?
- [ ] Did I discuss expiration and failure?
- [ ] Can I explain my ID strategy trade-off?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support custom aliases and prevent unsafe alias reuse after deletion.

**Score this design using the 40-point rubric.**


---

**Progress:** Session 27/46  
**Next:** Session 28
