# Session 32 — Design Twitter / Instagram News Feed

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice fan-out, ranking, feed caching, pagination, graph access, and celebrity/hot-user handling.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: follow users, create posts, fetch home feed; decide ranking/chronological scope.
- Graph/follow relationship storage.
- Fan-out on write versus fan-out on read.
- Hybrid approach for normal users versus celebrities.
- Feed/timeline cache and persistence.
- Ranking/pagination/freshness.
- Deletion/privacy propagation.
- Hot users and burst traffic.
- Eventual consistency/repair of missing feed entries.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design chronological feed first, then introduce ranking only if needed.
- [ ] Change request: celebrity live posts during a traffic spike.
- [ ] Compare fan-out-on-write and fan-out-on-read using concrete follower counts.

## **Must Remember for the Interview**

- **Fan-out strategy should come from follower distribution and read/write workload.**
- **Celebrities make fan-out-on-write expensive and create hot keys.**
- **Feed entries can be derived data; the post store remains authoritative.**
- **Cursor pagination is usually a better fit than deep offsets for changing feeds.**
- **Derived feeds need repair/rebuild behavior.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Fan-out-on-write → fast reads, expensive post writes. Fan-out-on-read → cheap writes, expensive reads.**
- **Hybrid is common: precompute normal-user feeds, fetch celebrity posts at read time.**
- **Cache timelines but preserve authoritative post data.**
- **Handle delete/privacy changes in derived feeds.**
- **Discuss ranking/freshness only to required depth.**

## Self-Test Before Marking This Session Complete

- [ ] Did I justify fan-out strategy?
- [ ] Did I discuss celebrity/hot-user behavior?
- [ ] Did I identify source of truth vs derived feed?
- [ ] Did I handle pagination?
- [ ] Did I discuss feed repair/deletion?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support celebrity live posts and sudden traffic spikes.

**Score using the 40-point rubric.**


---

**Progress:** Session 32/46  
**Next:** Session 33
