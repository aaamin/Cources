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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 30 — Design a Notification Service
