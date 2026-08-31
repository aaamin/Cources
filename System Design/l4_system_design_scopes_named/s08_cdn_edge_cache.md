# Session 8 — CDN & Edge Caching

## 1. Must Learn

### CDN purpose & edge locations
- **Understand:** Understand serving cacheable content near users to reduce latency, origin load, bandwidth, and cost.
- **Decision/trade-off:** Freshness/control vs cache efficiency and global performance.

### Origin vs edge
- **Understand:** Know cache misses go to origin and popular cacheable content should be absorbed at the edge.
- **Decision/trade-off:** Origin simplicity vs origin overload on misses/invalidations.

### Cache keys & TTL
- **Understand:** Understand which request attributes define an object and how long edge copies remain valid.
- **Decision/trade-off:** Hit rate vs staleness; too many key variants reduce caching.

### Invalidation/purge
- **Understand:** Know TTL alone may be insufficient for urgent changes.
- **Decision/trade-off:** Freshness vs operational complexity and purge propagation.

### Static vs dynamic/private content
- **Understand:** Know static/public assets are easiest to cache; personalized/sensitive responses require care.
- **Decision/trade-off:** Performance vs privacy/correctness.

### Origin protection
- **Understand:** Understand CDN can shield origin from traffic spikes, but miss storms or bypass paths can still overload it.
- **Decision/trade-off:** Availability/cost savings vs dependency on CDN behavior.

## 2. Should Know

- Signed URLs for controlled access to private media.
- Cache-Control concepts at a high level.
- Regional origin strategy when users are globally distributed.

## 3. Recognition Only

- Edge compute
- Origin shield/tiered caching

## 4. Important Comparisons

- CDN cache vs application/distributed cache.
- Static/public vs dynamic/personalized content.
- TTL expiration vs explicit purge.
- Public cacheability vs signed/private access.

## 5. Important Interview Questions

1. What content is safe and useful to cache at the edge?
2. What should the cache key include?
3. How stale can this content be?
4. How do urgent updates invalidate edge copies?
5. What happens if many objects miss simultaneously?
6. When should access use signed URLs?

## 6. Common Interview Mistakes

- **Caching personalized data publicly** → Make cache scope and privacy explicit.
- **Ignoring cache-key dimensions** → Wrong keys can leak or serve incorrect variants.
- **Assuming CDN removes origin need** → Origin still handles misses and must survive failures/spikes.
- **Very short TTL everywhere** → This can destroy hit rate and origin protection.
- **No invalidation strategy** → Define TTL and purge behavior.

## 7. Communication

### Important Vocabulary

CDN, edge location, origin, cache key, TTL, purge, invalidation, cache-control, signed URL, origin protection, hit rate

### Useful Interview Phrases

- “This asset is public and immutable enough to cache aggressively at the edge.”
- “The cache key must include the dimensions that change the response.”
- “The CDN reduces origin load, but I still need to protect the origin during miss storms.”

### Important Questions to Ask the Interviewer

- **Question:** “Is the content public or user-specific?”  
  **Why it matters:** Determines whether shared edge caching is safe.
- **Question:** “How quickly must updates become visible?”  
  **Why it matters:** Determines TTL/purge strategy.
- **Question:** “Is media delivery a major bandwidth cost?”  
  **Why it matters:** Determines CDN importance.

## 8. ⭐ Must Remember

1. CDNs move cacheable content closer to users.
2. Cache keys define correctness.
3. TTL trades freshness for hit rate.
4. Use purge when urgent freshness is required.
5. Do not publicly cache sensitive personalized content.
6. Protect the origin from miss storms and bypass traffic.

## 9. Study Priority

1. Study first: CDN/origin/edge and cacheability.
2. Study next: keys, TTL, invalidation.
3. Finish with: signed access and origin protection.

## 10. Revision Checklist

- [ ] Explain when a CDN helps.
- [ ] Design a safe cache key and TTL.
- [ ] Explain purge/invalidation.
- [ ] Distinguish public vs private content.
- [ ] Discuss origin overload on misses.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
