# Session 09 — Caching Fundamentals

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn where caches fit, the main read/write patterns, and how to define a cache policy rather than saying 'add Redis'.

## What You Need to Read / Learn

- Local in-process cache versus shared/distributed cache.
- Cache-aside: application checks cache, falls back to source of truth, populates cache.
- Read-through conceptually.
- Write-through: update cache as part of write path.
- Write-behind/write-back: async persistence and its durability/consistency risks.
- TTL and expiration.
- Eviction: LRU/LFU concepts and memory pressure.
- Cache hit ratio and what it means operationally.
- Source of truth and stale-data tolerance.

## What You Need to Do

- [ ] Create a cache policy for product details: key, value, TTL, invalidation, source of truth.
- [ ] Create a cache policy for user sessions and compare it to product-data caching.
- [ ] Explain whether to cache a frequently updated bank balance.

## **Must Remember for the Interview**

- **Always name the source of truth. A cache is usually derived data.**
- **Cache policy = key + value + TTL + invalidation + miss behavior + stale-data tolerance.**
- **Caching trades freshness and complexity for latency/capacity.**
- **Write-behind can lose acknowledged writes unless durability is designed carefully.**
- **A high cache hit rate can dramatically reduce database load.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Cache-aside is a common default for read-heavy derived data.**
- **TTL bounds staleness but does not guarantee instant invalidation.**
- **Local cache is fast but duplicated/inconsistent across instances.**
- **Distributed cache provides shared state but is another dependency.**
- **Do not cache data merely because Redis exists.**

## Self-Test Before Marking This Session Complete

- [ ] Can I describe cache-aside without notes?
- [ ] Can I define a complete cache policy?
- [ ] Can I explain write-through vs write-behind?
- [ ] Can I say when not to cache?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 9/46  
**Next:** Session 10
