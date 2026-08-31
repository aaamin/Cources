# Session 33 — Design Search Autocomplete

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice prefix retrieval, ranking, freshness, caching, and asynchronous index-building.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: top suggestions for a prefix; latency target; global vs personalized; freshness.
- Data sources: search/query logs or content corpus.
- Prefix structures: trie/sorted prefix index/search-engine approach at conceptual depth.
- Precomputed top-K suggestions per prefix.
- Ranking by frequency, recency, locale, optional personalization.
- Update pipeline and indexing lag.
- Caching popular prefixes.
- Index rebuild, versioning, and safe cutover.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design a non-personalized top-10 autocomplete first.
- [ ] Change request: support trending and personalized suggestions.
- [ ] Explain an index rebuild without serving a half-built index.

## **Must Remember for the Interview**

- **Autocomplete is usually a read-latency problem with asynchronous ranking/index updates.**
- **Precomputing top-K per prefix trades storage/update work for very fast reads.**
- **Freshness and ranking can be eventually consistent.**
- **Use versioned indexes and atomic cutover for rebuilds.**
- **Personalization should not destroy the ability to cache common/global results.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Prefix → retrieve candidate suggestions → rank → return top-K.**
- **Keep the serving path simple and fast.**
- **Build/update indexes asynchronously.**
- **Cache hot prefixes.**
- **For personalization, combine global candidates with lightweight user/context reranking.**

## Self-Test Before Marking This Session Complete

- [ ] Did I define latency/freshness requirements?
- [ ] Did I choose a prefix/index approach?
- [ ] Did I separate offline/update pipeline from serving?
- [ ] Did I discuss ranking and caching?
- [ ] Did I handle index rebuild/cutover?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Add trending queries and personalization.

**Score using the 40-point rubric.**


---

**Progress:** Session 33/46  
**Next:** Session 34
