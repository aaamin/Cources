# Session 13 — Data Modeling, Access Patterns & Read Models

## Outcome

You should be able to model data from reads/writes/invariants rather than from technology preference, identify source-of-truth entities, decide what to normalize or denormalize, create read models/precomputed views when justified, and state consistency/retention requirements.

## Start With Access Patterns

Before drawing tables/collections, write the important operations.

Example social app:

```text
Create post
Fetch post by ID
Fetch latest posts from followed users
Like/unlike post
Get like count
Follow/unfollow user
```

Now ask:
- key lookups?
- range scans?
- ordering?
- uniqueness?
- high write rate?
- high fan-out?
- required consistency?

The model follows these needs.

## Entities vs Access Paths

Entity:
```text
Post(id, author_id, body, created_at)
```

Access path:
```text
latest posts by author
feed for user
posts by hashtag
```

One entity can have multiple derived access paths.

Do not duplicate source-of-truth data casually, but duplicate/precompute when read performance requires it.

## Ownership

Every important piece of data should have a clear owner/source of truth.

Example:

```text
User Service owns user profile
Order Service owns order state
Search Index derives searchable representation
Feed Cache derives feed entries
```

If two systems both “own” the same mutable field, conflict/reconciliation becomes unclear.

## Model Invariants

Examples:
- email unique;
- seat sold at most once;
- wallet cannot spend unavailable balance;
- message ID unique within conversation;
- one coupon redemption per user.

Invariants often determine:
- transaction boundary;
- partition key;
- storage choice;
- consistency level.

## Normalized Write Model

For relational domains, normalized source of truth can be easy to reason about.

Example:
```text
User
Post
Follow
Like
```

Writes update authoritative rows/edges.

## Denormalized Read Model

A read model is optimized for a specific query.

Example feed:
```text
UserFeed(
  user_id,
  rank/time,
  post_id,
  author_snapshot,
  preview
)
```

The data is duplicated, but reads become fast.

Question:
- how is read model updated?
- can it lag?
- how is it rebuilt?

This turns denormalization into an explicit system rather than hidden inconsistency.

## Precomputation / Materialized Views

If a read is expensive and common, compute earlier.

Examples:
- daily analytics aggregates;
- trending topics;
- follower feed entries;
- product search index;
- dashboard counters.

Trade-off:

```text
More work/storage on write or background processing
          ↕
Faster reads
```

## Derived Data Must Be Rebuildable

A useful design principle:

If possible:
```text
Source of truth → event/change → derived model
```

If the feed cache/search index is corrupted, rebuild from authoritative data/events.

Not every derived dataset is easily rebuildable at huge scale, but the ownership principle remains useful.

## Retention

Modeling is affected by retention.

Example messages:
- retain forever;
- delete after 30 days;
- legal/audit archive;
- soft delete vs hard delete.

Retention influences:
- partitioning by time;
- cold storage;
- deletion workflow;
- index size.

## Ordering

If data must be ordered, define:
- by timestamp?
- sequence number?
- per user/conversation/global?
- tie-breaking?

“Messages are ordered” is incomplete.

Global ordering is much harder than per-conversation ordering.

## Uniqueness

Define scope:

```text
username → globally unique
message sequence → unique per conversation
external payment ID → globally unique for provider/account
```

Scoped uniqueness can affect key/index design.

## Relationship Cardinality

Examples:
- user → many posts;
- user ↔ many followers (many-to-many);
- event → many seats.

Cardinality matters for storage and queries.

A user with 500M followers is qualitatively different from an ordinary user. Model skew/hot entities.

## Example — Notification Preferences

Requirements:
- user can enable/disable channels per notification type;
- notification workers read preference frequently;
- updates are rare;
- strong immediate revocation may be required for some legal preference.

Source model:
```text
Preference(user_id, notification_type, channel, enabled, version)
```

Possible read cache:
```text
prefs:{user_id} → compact preference object
```

On update:
- transaction updates source;
- invalidate cache;
- for compliance-critical opt-out, worker may verify fresh version or propagate invalidation quickly.

Access pattern determines cache/data shape.

## Example — News Feed

Source of truth:
```text
Post
Follow
```

Derived read model:
```text
FeedEntry(user_id, post_id, score/created_at)
```

Fan-out workers populate FeedEntry.

Advantages:
- fast feed reads.

Costs:
- huge write amplification for users with many followers;
- eventual consistency;
- celebrity problem.

This leads naturally to hybrid fan-out later.

## Modeling Checklist

Before storage technology:

1. Core entities.
2. Core reads.
3. Core writes.
4. Invariants.
5. Ordering.
6. Uniqueness scope.
7. Cardinality/skew.
8. Consistency per operation.
9. Retention.
10. Derived/precomputed data.
11. Rebuild path.
12. Partition-local operations.

## Small Design Drills

1. Why should a feed cache not be treated as the source of truth for posts?
2. A query needs “latest 20 messages in conversation.” What ordering/key information matters?
3. Why can precomputed read models improve latency?
4. What is the cost of denormalization?
5. Why should you model a 100M-follower celebrity explicitly as skew?
6. What is the first thing to list before choosing SQL or NoSQL?

<details>
<summary>Answer key</summary>

1. It is derived/evictable; authoritative Post storage should survive cache loss.
2. Conversation ID partition/access key plus timestamp/sequence with deterministic tie-breaker.
3. Expensive joins/aggregation/fan-out work is moved earlier/background.
4. Extra storage, write/update propagation, eventual consistency, rebuild complexity.
5. Fan-out, hot keys, and storage/update patterns differ drastically from average.
6. Access patterns, invariants, ordering/consistency requirements.

</details>

## Common Interview Mistakes

- Designing schema from object classes without listing queries.
- Choosing database before modeling access patterns.
- Treating duplicated data as automatically bad.
- Treating all duplicated data as authoritative.
- Ignoring rebuild/reconciliation of derived data.
- Ignoring skew/cardinality.
- Saying “eventual consistency is fine” without specifying which field/operation.
- Undefined ordering semantics.

## Must Remember

- **Access patterns drive data modeling.**
- **Identify the source of truth.**
- **Invariants determine transaction/consistency needs.**
- **Read models can be intentionally denormalized.**
- **Precompute common expensive reads when justified.**
- **Derived data needs update and ideally rebuild paths.**
- **Define ordering and uniqueness scope.**
- **Model hot/skewed entities explicitly.**
- **Retention is part of the data model.**

## Interview Revision Summary

Write this before database choice:

```text
Entities
Reads
Writes
Invariants
Ordering
Uniqueness
Cardinality/skew
Consistency
Retention
Derived views
Partition-locality
```

## Explain Without Notes

Model the source-of-truth and derived read data for a basic Twitter-like feed. Explain what can be stale and what should remain authoritative.

## Completion Checklist

- [ ] I list access patterns first.
- [ ] I identify ownership/source of truth.
- [ ] I define invariants and ordering.
- [ ] I understand normalized vs read-optimized models.
- [ ] I can justify denormalization/precomputation.
- [ ] I consider skew and retention.
