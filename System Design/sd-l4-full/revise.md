# Master Revision Notes

Use this file for **quick revision only**. The lesson files are the learning material.

Recommended use:

```text
Before a lesson: read only that lesson.
After a lesson: use its Short Revision Note + Topics to Revise.
Weekly review: use this master sheet.
Before mocks: revisit weak categories, not every lesson.
```

## 01. System Design Interview Framework

**Revise:** framework; requirements; non-goals; estimation; API/data; HLD; flows; deep dive; failure; trade-offs; closing.

## 02. Back-of-the-Envelope Estimation

**Revise:** DAU/QPS; peak factor; read/write ratio; storage; bandwidth; concurrency; skew; decision consequence.

## 03. Networking Fundamentals

**Revise:** DNS; TCP/UDP; TLS; HTTP; connection reuse; latency; timeouts; cross-region round trips.

## 04. Client–Server Architecture & Request Lifecycle

**Revise:** request lifecycle; critical path; stateless/stateful; durable/ephemeral state; connection pools; async side effects.

## 05. Load Balancing

**Revise:** L4/L7 LB; algorithms; health/readiness; draining; sticky sessions; global vs regional routing.

## 06. Horizontal & Vertical Scaling

**Revise:** vertical/horizontal; stateless compute; bottlenecks; autoscaling signals; headroom; scale vs availability.

## 07. Proxy, Reverse Proxy & API Gateway

**Revise:** forward/reverse proxy; API gateway; TLS termination; edge auth; routing; rate limits; BFF.

## 08. CDN & Edge Caching

**Revise:** CDN; origin; edge cache; TTL/versioning; signed URLs; origin shield; private content.

## 09. Caching Fundamentals

**Revise:** cache-aside; read/write-through; write-behind; TTL; invalidation; eviction; hit rate; source of truth.

## 10. Advanced Caching Problems

**Revise:** stampede; single-flight; stale-while-revalidate; hot key; penetration; negative cache; warmup; outage fallback.

## 11. SQL & Relational Databases

**Revise:** relational model; PK/FK; normalization; constraints; transactions; invariants; denormalization.

## 12. Database Indexing

**Revise:** primary/secondary/composite index; prefix/order; selectivity; covering index; write amplification.

## 13. Data Modeling & Access Patterns

**Revise:** entities; access patterns; ownership; invariants; state machine; ordering/cursor; retention; derived views.

## 14. NoSQL Databases

**Revise:** key-value; document; wide-column; denormalization; consistency options; search index; polyglot cost.

## 15. Database Replication & Failover

**Revise:** leader/follower; sync/async; lag; read-after-write; read replicas; failover; split brain; failure domains.

## 16. Partitioning & Sharding

**Revise:** hash/range/geo partition; shard key; hot partitions; scatter-gather; logical shards; resharding.

## 17. Consistent Hashing

**Revise:** modulo remap; hash ring; virtual nodes; membership; replication placement; hot-key limitation.

## 18. Consistency, CAP & Quorums

**Revise:** strong/eventual; read-after-write; monotonic; CAP partition choice; quorum; conflicts; convergence.

## 19. Message Queues & Async Processing

**Revise:** producer/consumer; ack/lease; at-most/at-least once; idempotency; retry; DLQ; ordering; queue age.

## 20. Pub/Sub, Streams & Kafka Concepts

**Revise:** topic/partition; offsets; consumer groups; retention; replay; event versioning; consumer lag; batch/stream.

## 21. Reliability, Overload & Failure Isolation

**Revise:** timeout; retry/backoff/jitter; circuit breaker; bulkhead; backpressure; admission; shedding; degradation.

## 22. Idempotency, Transactions, Concurrency & Distributed Workflows

**Revise:** race; optimistic/pessimistic; idempotency; leases/fencing; Saga; outbox/CDC; reconciliation; dual write.

## 23. API & Event Contract Design

**Revise:** REST resources; methods; cursor pagination; errors/retries; idempotency key; versioning; gRPC; webhook/events.

## 24. Real-Time Communication

**Revise:** polling; long polling; SSE; WebSocket; gateway; heartbeat; presence; reconnect/resume.

## 25. Object Storage, Media, Multi-Region & Disaster Recovery

**Revise:** object/metadata; multipart; checksum; signed URL; multi-AZ/region; active/passive/active; RPO/RTO; backup.

## 26. Observability, SLOs, Deployment, Security & Privacy

**Revise:** metrics/logs/traces; p99; SLI/SLO/SLA; canary; migration; authn/authz; encryption; PII; abuse.

## 27. Design a URL Shortener

**Revise:** URL shortener: ID/alias uniqueness; redirect cache; hash sharding; expiry; viral key; async analytics.

## 28. Design a Distributed Rate Limiter

**Revise:** rate limiter: window/token bucket; atomic state; keys; hierarchy; global quota; fail-open/closed.

## 29. Design Pastebin

**Revise:** Pastebin: ID; expiration; metadata/blob split; large upload; cache/CDN; privacy/abuse.

## 30. Design a Notification Service

**Revise:** notifications: priority isolation; queues/workers; preferences; provider adapters; retry/DLQ; dedupe; campaigns.

## 31. Design WhatsApp / Messenger

**Revise:** chat: connection gateway; durable messages; per-conversation order; offline cursor; presence; groups; media.

## 32. Design Twitter / Instagram News Feed

**Revise:** feed: fan-out write/read; hybrid celebrity; derived feed; ranking/cache; deletion; fan-out replay.

## 33. Design Search Autocomplete

**Revise:** autocomplete: prefix top-K; trie/index; precompute; ranking; trends; personalization; hot prefix; version cutover.

## 34. Design Dropbox / Google Drive

**Revise:** Drive: metadata/blob; stable file ID; chunk upload; versions; sync cursor; conflicts; tombstone/GC; sharing.

## 35. Design YouTube / Video Platform

**Revise:** video: upload; processing DAG; renditions; segments/manifest; ABR; CDN/origin; live event; async views.

## 36. Design a Distributed Web Crawler

**Revise:** crawler: frontier; canonicalization; dedupe; robots/politeness; host scheduler; leases; recrawl/priority.

## 37. Design Uber / Ride Matching

**Revise:** Uber: location ingest; geo index; nearby/refine; freshness; matching; trip state; assignment; regional hot cells.

## 38. Design Ticketmaster

**Revise:** Ticketmaster: inventory invariant; holds; conditional write; payment idempotency; reconciliation; waiting room.

## 39. Design a Metrics / Logging Platform

**Revise:** metrics/logging: batch ingest; stream; raw archive; TS rollups; search; cardinality; retention; tenant isolation.

## 40. Design a Distributed Job Scheduler

**Revise:** scheduler: Job vs Execution; due index; atomic claim; queue; lease/fencing; idempotency; retry; recurrence.

## 41. Mock #1 — Read-Heavy Service

**Revise:** mock: read-heavy; source vs derived; cache outage; search/index freshness; hot reads.

## 42. Mock #2 — Realtime / Async Workflow

**Revise:** mock: realtime; connection/fan-out; ordering scope; moderation; retries/dedupe; hot event.

## 43. Mock #3 — Data-Heavy / Write-Heavy

**Revise:** mock: write-heavy; stream; partition; retention; replay; lag; noisy tenant.

## 44. Mock #4 — High Scale with Skew

**Revise:** mock: skew; windows; top-K; hot-key aggregation; event-time lag; regional trends.

## 45. Mock #5 — Correctness / Concurrency

**Revise:** mock: invariant; concurrency; holds; payment ambiguity; idempotency; Saga/reconciliation.

## 46. Final Unseen FAANG-Style Mock

**Revise:** final: scope control; invariants; checkout workflow; multi-region consistency; failover; overload; closing.
