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


## Detailed reference design

### What the feed system actually does

There are two primary operations:

1. user creates a post;
2. follower reads a home timeline.

The hard problem is converting the **follow graph + posts + ranking** into a low-latency personalized list at enormous read volume.

### Data ownership

```text
Post Store       = source of truth for posts
Follow Graph     = source of truth for relationships
Feed Store/Cache = derived materialized view
Ranking features = derived
```

This source/derived distinction makes repair easier. If fan-out misses an entry, the feed can be rebuilt from posts + graph.

### Fan-out on write

When Alice posts:

```text
Post Service → persist post
      ↓
PostCreated event
      ↓
Fan-out workers
      ↓
append post_id to follower feed inboxes
```

Read is fast: fetch one user's prepared list.

Cost: author with 50M followers creates 50M writes/tasks.

### Fan-out on read

When Bob opens feed:

1. fetch Bob's followees;
2. fetch recent posts from each;
3. merge/rank.

Write is cheap, but read is expensive for users following many accounts and repeated feed opens.

### Hybrid strategy

Use write fan-out for ordinary authors. For high-follower celebrities, do not prewrite to every follower. At read time, merge recent celebrity posts into the precomputed feed.

This is a classic workload-aware hybrid.

### Feed representation

Do not copy complete post bodies into every feed entry. Store compact references:

```text
(user_id, rank/time, post_id)
```

Read path batch-fetches post objects from post cache/store. This reduces fan-out storage and makes edits/deletes easier.

### Ranking

Start chronological if ranking is not the main prompt. If ranking is required, separate candidate generation from ranking:

```text
candidate post IDs
  ↓
feature lookup
  ↓
ranking service/model
  ↓
top N
```

Ranking can fail; fallback to chronological/precomputed ordering keeps feed available.

### Pagination

Use cursor based on rank/time + stable ID rather than offset. Feed changes while user scrolls, so exact no-duplicate/no-missing semantics may require a feed-generation version/snapshot or accepting minor drift.

### Cache

Cache hot feed pages and post objects. A user's first page is much hotter than page 100. Precompute/cache top N and generate deeper history on demand.

## Failure walkthrough

### Fan-out worker lag

New posts appear late. Since feed is derived, workers can retry/replay. Monitor event-time lag. Critical post creation should still succeed if fan-out is delayed.

### Celebrity surge

Celebrity posts remain read-time merged, avoiding tens of millions of writes. Cache celebrity recent-post lists heavily.

### Deleted post

Post Store marks deleted/tombstone. Feed references may remain temporarily, but read path filters missing/deleted post. Async cleanup removes stale feed entries.

### Ranking outage

Serve chronological/previous cached feed. This is graceful degradation.

## Interviewer follow-ups

### “What if users follow 10,000 accounts?”

Precomputed feed limits read fan-in. On-read celebrity merge is bounded. Follow graph and feed candidate generation may cap/look at recent active accounts rather than scan all history.

### “What if a new user has no feed?”

Use onboarding/trending recommendations as separate candidate source. The core follow feed can be empty.

### “How do you guarantee a post appears?”

If product requires stronger semantics for own posts, inject user's own post directly/read-after-write. General follower feeds can be eventually consistent.

## Common interview mistakes

- Fan-out on write for celebrity with no exception.
- Copying full post body into millions of feed rows.
- Feed cache treated as source of truth.
- No repair/replay when fan-out fails.
- Offset pagination on highly changing feed without discussion.
- Ranking made critical dependency with no fallback.

## Short revision note

**Feed pattern:** source posts + follow graph → materialized follower feeds for ordinary authors + read-time merge for celebrity/hot authors → cache + ranking. Feed is a derived repairable view.

## Topics to revise

- [ ] fan-out write/read
- [ ] hybrid celebrity strategy
- [ ] source vs derived feed
- [ ] feed entry references
- [ ] ranking fallback
- [ ] cursor pagination
- [ ] fan-out lag/replay
- [ ] hot users

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll separate Post and Follow truth from the derived home-feed view. The main design choice is fan-out on write vs read. I’ll use workload distribution to justify a hybrid where ordinary authors precompute follower feeds and celebrity posts merge at read time.

## How the design evolves at 10×

At 10× reads, first-page feed cache and post-object cache become critical. At 10× celebrity activity, expand read-time candidate strategy. At 10× graph size, partition follow graph/user feeds and keep fan-out replayable.

## Quick revision flashcards

**Feed source of truth?**  
Posts + follow graph; materialized feed is derived.

**Celebrity problem?**  
Fan-out-on-write creates millions of writes; merge celebrity posts on read.

**Delete post?**  
Source tombstone + read filtering + async feed cleanup.

**Ranking down?**  
Fallback to chronological/cached feed.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 33 — Design Search Autocomplete
