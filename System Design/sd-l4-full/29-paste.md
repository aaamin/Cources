# Lesson 29 — Design Pastebin

**Phase:** Guided Design  
**Session:** 29/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Create a text paste.
- Read it by short ID.
- Support expiration.
- Handle large payloads without overloading the metadata database.

## 2. Scale and workload shape

Usually read-heavy. Estimate paste creation rate, average/maximum paste size, retention, and read/write ratio. Small text can live in a DB; large or numerous payloads push toward object storage and CDN delivery.

## 3. API / contract surface

```http
POST /v1/pastes
GET  /v1/pastes/{id}
```

Create accepts content or an upload reference, visibility, and expiration. It returns an ID/URL.

## 4. Data model

```text
Paste(
  paste_id PRIMARY KEY,
  object_key or content,
  owner_id?,
  created_at,
  expires_at,
  size,
  visibility
)
```

Metadata is queryable; large bodies can live as objects.

## 5. High-level architecture

```text
Client
  ↓
Paste API
  ├─ Metadata DB
  ├─ Cache
  └─ Object Storage
         ↑
       CDN (public popular pastes)
```

Read checks metadata/expiry, then returns content or a signed/object URL.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### ID generation

Random IDs reduce enumeration. If the paste is private, real authorization is still required; “hard-to-guess ID” is not sufficient security.

### Expiration

Reads enforce expiration immediately. Background deletion can remove DB rows/blobs later, so cleanup delays do not violate user-visible expiry.

### Large pastes

Direct upload to object storage avoids buffering megabytes through app servers. Store content hash/checksum for integrity if useful.

## 7. Failure modes and recovery

- Cleanup worker down: expired data consumes storage but reads still reject it.
- Cache stale after delete: invalidate or use bounded TTL.
- Blob store unavailable: metadata is readable but content is not; return controlled error.
- Abuse: enforce payload limits, quotas, and scanning/moderation if in scope.
- Duplicate create retry: optional idempotency prevents duplicate records.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

A single SQL table containing text is the simplest first version. Split metadata/blob storage only when object size, cost, or scale justifies it.

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

### Small-paste first version

For modest scale, a relational table containing the body is completely reasonable. One service + DB can support create/read/expiration with an index on ID. This is a good example of resisting unnecessary object storage and microservices until size/traffic require them.

### When to split blobs

Suppose the maximum paste becomes 100 MB and users upload many binary/text artifacts. Large rows can make backups, DB cache, replication, and reads inefficient. Store content in object storage and keep only metadata/reference in the DB. This follows the same control-plane/data-plane split used in Drive and video.

### Read flow

```text
GET /p/{id}
→ lookup metadata/cache
→ reject if expired/private without permission
→ return small DB body OR signed/object/CDN content
```

A public immutable paste can be CDN-cached. A private paste needs access control before issuing a signed URL.

### Expiration semantics

User-visible expiration and physical deletion do not need to occur at exactly the same moment. Check `expires_at` on reads so the paste becomes inaccessible immediately; asynchronously delete data later. This separates correctness from cleanup efficiency.

### Common interview mistakes

- Assuming unguessable IDs replace authorization.
- Performing synchronous full-storage deletion before returning every delete request.
- Using a DB for huge blobs without considering backup/replication impact.
- Forgetting storage abuse and per-user quota.
- Caching private data with shared public keys.

### Reusable patterns learned

TTL/expiration, metadata + blob split, direct object delivery, random identifiers, privacy vs obscurity, and asynchronous garbage collection.


## Detailed reference design

### Why Pastebin is a useful interview problem

Pastebin combines several basic patterns without too much domain complexity:

- short-ID creation;
- read-heavy lookup;
- TTL/expiration;
- variable-size payload storage;
- caching/CDN;
- abuse/security.

The key is to keep the first design simple, then separate metadata and large blobs only when size justifies it.

### Requirements

Core:

- create text paste;
- retrieve by ID;
- optional expiration;
- optional private/unlisted visibility.

Useful non-functional goals:

- low read latency;
- durable writes;
- efficient expiration;
- maximum size policy.

### Data model

For small/medium payloads:

```text
Paste(
  id,
  owner_id?,
  body,
  created_at,
  expires_at,
  visibility
)
```

This can live entirely in SQL initially.

For large paste support:

```text
Paste metadata DB
  id → object_key, size, hash, expiration

Object store
  object_key → body
```

This prevents huge text blobs from bloating hot metadata indexes/pages.

### Create flow

```text
Client → Paste API
          ├ validate size/type
          ├ generate ID
          ├ write body or object
          └ write metadata
```

If object storage is used, avoid publishing metadata as readable until object upload is successfully committed. A state like `UPLOADING → READY` helps prevent dangling references.

### Read flow

```text
GET /p/{id}
  ↓
cache metadata/content
  ├ hit → return
  └ miss → metadata DB → object store (if used) → cache
```

Public immutable pastes can be CDN-cached. Private pastes require authorization-aware delivery and should not leak through shared cache keys.

### Expiration

Correctness check:

```text
if expires_at <= now → return not found/expired
```

Cleanup can be lazy/background. You do not need a timer per paste. Use a periodic scan/index by expiration bucket, TTL feature of storage, or lifecycle policy for object storage.

### Very large pastes

For a 100 MB paste, proxying through application servers wastes bandwidth. Use direct object upload with a signed URL after the API creates an upload record. The client completes upload; API marks paste ready.

### Security/abuse

Public Pastebin-like systems are abuse targets. Mention:

- size/rate limits;
- private/unlisted semantics;
- malware/spam scanning if in scope;
- enumeration risk;
- retention/moderation policy.

Do not let abuse controls dominate unless interviewer asks.

## Failure walkthrough

- metadata write succeeds, blob upload fails → paste remains non-ready and cleanup removes abandoned object/record;
- cleanup job stops → expired content may occupy storage but reads still enforce expiration;
- cache stale after deletion → invalidate and short safety TTL;
- object store unavailable → metadata API works but content reads fail/degrade;
- duplicate create retry → idempotency key can return same paste.

## Interviewer follow-ups

### “Would you use SQL or NoSQL?”

SQL is a perfectly good start because the data model is simple and QPS may be modest. A key-value store becomes attractive when ID lookup dominates at very large scale. Object storage is driven by payload size, not user count alone.

### “How do you generate IDs?”

Random base62/UUID-derived IDs or distributed sequences. If unlisted links rely on unguessability, use enough entropy. Unlisted is not the same as authenticated private access.

### “What happens after expiration?”

Reads reject immediately based on metadata/TTL; physical deletion can lag. This separates user-visible correctness from background reclamation.

## Common interview mistakes

- Object storage for tiny text from day one with no reason.
- One timer per paste for expiration.
- Unlisted treated as secure authorization.
- Large blob stored in a hot indexed row without trade-off.
- Metadata points to incomplete upload.
- No size/rate limits.

## Short revision note

**Pastebin pattern:** short ID + key lookup + expiration + optional metadata/blob split + cache/CDN + simple abuse controls.

## Topics to revise

- [ ] ID generation
- [ ] metadata vs blob
- [ ] expiration correctness vs cleanup
- [ ] cache/CDN
- [ ] direct large upload
- [ ] private vs unlisted
- [ ] idempotent create

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll keep V1 simple: paste metadata/body keyed by generated ID with expiration. If payloads are large, I’ll separate metadata from object storage. Reads are cacheable; expiration correctness is checked on read while background cleanup reclaims space.

## How the design evolves at 10×

At 10× bytes, object storage/CDN matter more than API compute. At 10× lookup QPS, cache popular content. At 10× create volume, partition metadata by paste ID and make cleanup/TTL distributed.

## Quick revision flashcards

**Expiration source?**  
Metadata/TTL checked at read; cleanup can lag.

**Large body?**  
Direct object upload/storage, not hot DB row/app proxy.

**Private vs unlisted?**  
Unlisted is hard-to-guess; private needs authorization.

**ID strategy?**  
Random/distributed unique ID with enough entropy for product policy.

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

**Next:** Lesson 30 — Design a Notification Service
