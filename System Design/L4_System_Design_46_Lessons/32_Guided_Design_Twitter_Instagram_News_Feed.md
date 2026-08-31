# Session 32 — Guided Design — Twitter / Instagram News Feed

## Interview Prompt

> Design a Twitter/Instagram-style home feed. Users follow others and see recent/relevant posts from followed accounts.

Change request:
> A celebrity with 100M followers posts during a major live event.

Attempt for **40–50 minutes** before continuing.

---

# STOP — Attempt First

Cover:
- follow graph;
- post storage;
- feed generation;
- fan-out on write/read;
- hybrid fan-out;
- ranking;
- cache;
- pagination;
- hot users;
- consistency/freshness.

---

# Reference Reasoning

## 1. Requirements

Core:
- create post;
- follow/unfollow;
- home feed;
- pagination;
- recent posts;
- optional ranking.

Clarify:
- chronological or ranked?
- maximum freshness target?
- private accounts?
- delete/edit propagation?
- media?
- likes/comments out of scope?

Non-functional:
- feed read low latency;
- high availability;
- eventual feed freshness acceptable;
- posts themselves durable;
- celebrities/hot users create skew.

## 2. Estimates

Assume:
- 100M DAU;
- 20 feed opens/day → 2B feed reads/day ≈ 20k avg RPS, much higher peak;
- 50M posts/day;
- follower distribution is highly skewed.

The most important estimate is not average follower count—it is tail users with tens/hundreds of millions of followers.

## 3. Data Model

Source of truth:

```text
User
Post(post_id, author_id, body/media_ref, created_at, status)
Follow(follower_id, followee_id)
```

Derived feed model:

```text
FeedEntry(
  user_id,
  rank_key / created_at,
  post_id,
  author_id
)
```

Post body remains authoritative in Post store.

FeedEntry can be rebuilt/updated asynchronously.

## 4. API

```http
POST /posts
GET /feed?cursor=...
POST /users/{id}/follow
DELETE /users/{id}/follow
```

Cursor for feed should encode stable order/ranking position.

## 5. Fan-Out on Write

When Alice posts:

```text
Post Service
   ↓
followers(Alice)
   ↓
write post_id into each follower feed
```

Advantages:
- feed reads fast;
- work precomputed.

Costs:
- write amplification;
- celebrity post fan-out huge;
- inactive users waste work.

Good for normal users with bounded follower counts.

## 6. Fan-Out on Read

Do not precompute per-user feed.

At read:
1. load followees;
2. fetch recent posts from each;
3. merge/rank.

Advantages:
- cheap writes;
- no 100M-entry celebrity fan-out.

Costs:
- expensive/slow feed reads;
- many sources;
- difficult at large follow counts.

Good for celebrity/high-fanout authors as one component.

## 7. Hybrid Fan-Out

Common strong answer:

```text
Normal authors → fan-out on write
Celebrities → do not fan-out to everyone
```

At feed read:
- read precomputed feed;
- fetch recent celebrity posts separately;
- merge/rank.

Threshold can be based on:
- follower count;
- activity;
- fan-out cost;
- current traffic.

This handles the celebrity problem.

## 8. High-Level Architecture

```text
Create Post:
Client → Post API → Post Store
                    ↓
                 Event Stream
                    ↓
              Fan-out Workers
                    ↓
                Feed Store

Read Feed:
Client → Feed API
           ↓
      Feed Cache/Store
           +
      Celebrity Post Read
           ↓
        Rank/Merge
           ↓
        Post Cache/Store
```

Media served separately through object storage/CDN.

## 9. Feed Store

Access:
```text
user_id → ordered list of post IDs/rank entries
```

Could use:
- sorted KV/wide-column;
- cache + durable derived store.

Keep bounded recent history:
- maybe last N feed entries;
- older history can be reconstructed or not supported depending requirements.

## 10. Ranking

Start simple:
```text
reverse chronological
```

If ranked:
- candidate generation;
- features;
- ranking service.

Do not turn system-design interview into ML architecture unless required.

Ranking can use:
- precomputed score;
- online score;
- hybrid.

Stable cursor must account for rank updates. A time-based feed is easier.

## 11. Follow/Unfollow Semantics

On follow:
- future posts appear;
- optionally backfill recent posts.

On unfollow:
- remove future fan-out;
- old entries may remain until filter/cleanup.

At read, validate relationship for sensitive/private content if needed.

Private account authorization cannot rely only on stale feed cache.

## 12. Delete/Edit

Delete:
- mark authoritative post deleted;
- invalidation event removes feed/cache/search entries;
- read path should not show deleted content even if derived feed entry lags.

One strategy:
- feed entry stores post ID only;
- final post fetch checks active state.

Cost: extra lookup, mitigated with cache.

Edit:
- post body updated once;
- feed entries need not duplicate full content if they reference post ID.

## 13. Feed Cache

Cache first page/user feed.

Key:
```text
feed:{user_id}:{version}
```

Challenges:
- frequent updates;
- user-specific;
- invalidation.

Often feed store itself is a precomputed read model; additional cache may be useful for hottest first page.

## 14. Hot Users

Celebrity post:
- avoid 100M synchronous writes;
- publish once;
- celebrity post store/cache replicated;
- merge at read.

Celebrity profile/post key may be hot:
- CDN for public media/content;
- distributed/local caches;
- replicas.

## 15. Backpressure

Fan-out workers may lag during event.

Monitor:
- queue/stream lag;
- oldest fan-out age.

If lag grows:
- prioritize fresh/high-value updates;
- autoscale workers;
- celebrity switch to fan-out-on-read;
- drop/rebuild low-priority derived feed work if safe.

Posts remain durable even if feed propagation lags.

## 16. Multi-Region

User/home-region approach:
- posts authoritative in author's/home region;
- events replicated;
- feeds generated regionally.

Cross-region followers may see post after replication delay.

If global freshness requirement is tight, distribute post events quickly but accept complexity.

## 17. Failure Scenarios

### Fan-out workers down
Posts still accepted; feed freshness degrades; backlog catches up.

### Feed store down
Fallback to on-read merge for limited subset or return degraded recent posts.

### Cache down
Derived feed store/backing DB sees more load; protect with rate limits/headroom.

### Event duplicated
Fan-out insertion idempotent on `(user_id, post_id)`.

## 18. Security/Privacy

- private accounts;
- blocks;
- deleted content;
- authorization;
- abuse/spam;
- scraping/rate limiting.

Do not serve private post solely because it is in stale feed cache.

## Interview Questions

1. Fan-out on write vs read?
2. What is celebrity problem?
3. How does hybrid work?
4. What happens when fan-out lags?
5. How do you delete a post from millions of feeds?
6. How do you paginate ranked feed?
7. Why not store full post body in every feed entry?
8. Which data is source of truth?

## Common Mistakes

- Fan-out every celebrity post to 100M users.
- Fan-out on read for every normal user without latency discussion.
- Feed store treated as post truth.
- No skew/follower-tail reasoning.
- Delete privacy ignored.
- Ranking overcomplicated.
- No idempotency in fan-out.
- No lag/backpressure monitoring.

## Must Remember

- **Feed is a derived read model.**
- **Fan-out on write trades expensive writes for cheap reads.**
- **Fan-out on read trades cheap writes for expensive reads.**
- **Hybrid handles celebrity skew.**
- **Post store remains authoritative.**
- **Fan-out lag means stale feed, not lost posts.**
- **Deletion/privacy must override stale derived entries.**
- **Idempotent feed-entry writes make replay safe.**

## Self-Score

Use the 40-point rubric. Redo fan-out/hot-user deep dive if scalability or trade-offs <3.
