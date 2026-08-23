# Lesson 32 — Design Twitter / Instagram News Feed

**Phase:** Guided Design  
**Session:** 32/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Users create posts.
- Users follow accounts.
- Home feed returns recent/ranked posts.
- Low read latency.
- Handle celebrity accounts and skew.

## 2. Scale and workload shape

Estimate posts/day, feed reads/day, average followers, and especially the heavy-tail follower distribution. A few accounts with millions of followers drive the architecture more than the average user.

## 3. API / contract surface

```http
POST /v1/posts
POST /v1/follows/{user_id}
GET  /v1/feed?cursor=...
```


## 4. Data model

```text
Post(post_id, author_id, created_at, content_ref)
Follow(follower_id, followee_id)
FeedEntry(user_id, rank_or_time, post_id)   # if materialized
```

Post store is source of truth; feed entries are derived.

## 5. High-level architecture

```text
Post Service → Post Store
      ↓
Fan-out Workers ──> Feed Store/Cache   (normal authors)

Celebrity posts ──> merged/ranked on read

Client → Feed Service → Feed Cache/Store + Post Store
```


Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Fan-out on write

On each post, write the post ID into follower feeds. Reads become fast, but high-follower authors create huge write amplification.

### Fan-out on read

Store posts once and compute a user's feed when requested. Writes are cheap but every feed read performs graph/read work.

### Hybrid

Materialize normal authors on write. For celebrities, keep posts separate and merge them when followers read. This follows the heavy-tail workload.

## 7. Failure modes and recovery

- Fan-out worker lag: feeds temporarily stale; replay from durable post/event log.
- Celebrity spike: avoid writing millions of feed rows synchronously.
- Deleted post: tombstone/check source so stale FeedEntry does not render.
- Ranking service down: fall back to chronological feed.
- Cache failure: protect feed store/database from sudden amplification.
- Follow/unfollow propagation lag: define expected freshness.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Precomputation trades write amplification/storage for read latency. Hybrid fan-out is valuable because real social graphs are highly skewed.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Why the graph shape matters

Most users have tens/hundreds of followers, while a tiny number have millions. Designing from the average follower count hides the hardest case. This is a classic heavy-tail workload: optimize the common path but add a special path for extreme nodes.

### Post creation flow

```text
1. Store canonical post.
2. Publish PostCreated event.
3. If normal author: workers write post ID into follower feed stores.
4. If celebrity: mark post for on-read merge rather than fan-out to millions immediately.
5. Invalidate/update author's own views as needed.
```

Fan-out can lag without losing the post because the canonical store/event is durable and replayable.

### Feed read flow

```text
user opens feed
→ fetch materialized feed IDs
→ fetch candidate celebrity posts
→ rank/merge
→ hydrate post objects (cache/batch)
→ return cursor page
```

Avoid one DB call per post; batch or cache hydration.

### Ranking and freshness

The feed store can hold candidate IDs, while a ranking service orders them using recency/relevance. If ranking fails, chronological order is a good graceful fallback. Deletions or privacy changes should be checked during hydration or propagated with tombstones so stale feed entries do not leak content.

### Common interview mistakes

- Fan-out every celebrity post to millions of followers synchronously.
- Recomputing the full feed from graph traversal on every read without scale reasoning.
- Forgetting post deletion/privacy after materialization.
- Fetching each post body with N+1 database requests.
- Treating ranking as source of truth.

### Reusable patterns learned

Fan-out-on-write vs read, hybrid heavy-tail strategy, materialized views, event replay, ranking fallback, pagination, and cache-friendly hydration.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 33 — Design Search Autocomplete
