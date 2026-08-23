# L4 System Design Coverage Audit

This file documents what the course intentionally covers and what it intentionally leaves outside the main 46-session system-design track.

## Verdict

The handbook now covers the major knowledge expected for a **generalist L4 / mid-level high-level system-design interview**:

- requirement framing and communication;
- capacity estimation and workload shape;
- networking, DNS/TCP/TLS/HTTP;
- load balancing, proxies, gateways, service discovery;
- vertical/horizontal scaling and stateless compute;
- CDN and caching, including pathological cache behavior;
- SQL, data modeling, indexes, B-tree/LSM/Bloom recognition;
- NoSQL plus key-value/document/wide-column/search/time-series/graph/object-store recognition;
- replication, failover, leader election/fencing, multi-leader recognition;
- partitioning, sharding, consistent hashing, distributed ID strategies;
- consistency, CAP, quorums, leaderless repair concepts;
- queues, Pub/Sub, event streams, replay, delivery semantics;
- retries, timeouts, backoff, circuit breaking, bulkheads, backpressure, load shedding;
- transactions, concurrency, idempotency, locks/leases/fencing;
- Saga, compensation, outbox, CDC, reconciliation, CQRS/event-sourcing recognition;
- REST/gRPC/webhooks, pagination, idempotency contracts;
- polling, SSE, WebSockets, presence and reconnects;
- object storage, large files/media, multi-region, RPO/RTO, cost;
- metrics/logs/traces, SLOs, deployment safety;
- authentication/authorization, tokens, OAuth/OIDC recognition, RBAC/ACL, privacy/abuse;
- search/autocomplete, geospatial systems, media pipelines, crawlers, metrics pipelines;
- real design practice for read-heavy, realtime, write-heavy, skew-heavy, and correctness-heavy systems.

## What is intentionally not a prerequisite

The course does not require implementation-level depth in:

- Raft/Paxos internals;
- CRDT mathematics;
- Byzantine consensus;
- database storage-engine implementation;
- Kubernetes internals/service-mesh administration;
- advanced distributed transaction protocols;
- advanced graph algorithms;
- full search-engine internals;
- ML system design.

These can matter for specialized infrastructure roles, but they are not efficient prerequisites for a general L4 system-design round.

## Important scope boundary

This is a **system-design/HLD course**. A company may separately test:

- low-level/object-oriented design;
- coding/data structures;
- behavioral/leadership;
- domain-specific frontend/mobile design.

Confirm the actual interview loop before the final mock phase. If the target round is explicitly LLD/object design, prepare that as a separate track rather than diluting the HLD curriculum.

## Final preparation rule

Do not measure readiness by topic count. Readiness requires repeated unseen performance:

1. clarify a vague problem;
2. build a simple complete design;
3. explain the data model and request flows;
4. identify real bottlenecks;
5. preserve correctness under failure/retry;
6. discuss alternatives and trade-offs;
7. adapt when the interviewer changes a requirement;
8. finish on time with a coherent summary.
