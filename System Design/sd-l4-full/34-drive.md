# Lesson 34 — Design Dropbox / Google Drive

**Phase:** Guided Design  
**Session:** 34/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Upload and download files.
- Synchronize changes across devices.
- Support version history and offline edits.
- Share files securely.

## 2. Scale and workload shape

Estimate active users, file count, average and maximum file size, uploaded bytes/day, sync events/day, and metadata read/write QPS. The architecture has two very different workloads: small metadata operations and large blob transfers.

## 3. API / contract surface

Control-plane APIs can be:

```http
POST /v1/files/init-upload
POST /v1/files/{id}/complete
GET  /v1/files/{id}/metadata
GET  /v1/changes?cursor=...
```

The service returns signed URLs so clients upload/download large bytes directly to object storage.

## 4. Data model

```text
File(file_id, owner_id, name/path, current_version, deleted_at?)
Version(file_id, version_id, manifest_ref, size, created_at)
Permission(file_id, principal, role)
Change(user_id, sequence, file_id, version_id, operation)
```

Chunks/objects live outside the relational metadata store.

## 5. High-level architecture

```text
                 ┌→ Metadata/Sync API → Metadata DB
Client ──────────┤
                 └→ Signed URL ↔ Object Storage
                                      ↓
                              Change/Event Stream
                                      ↓
                               Other Devices
```

Metadata commits establish file/version truth; object storage carries large bytes.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Resumable upload

Split large files into chunks, upload chunks independently, verify hashes/checksums, and commit a manifest only after all required chunks exist. Interrupted transfers resume only missing parts.

### Sync protocol

Each device keeps a change cursor/version. The server exposes changes since that cursor. Stable IDs and version numbers let devices dedupe and apply operations in order.

### Offline conflict

Two devices can edit the same old version. General binary files cannot always be merged safely, so preserve both/conflict-copy or use last-writer only when product semantics allow it.

### Deletion and garbage collection

Deletion may create a tombstone first. Blob/chunk cleanup happens later after retention and reference checks.

## 7. Failure modes and recovery

- Upload interrupted: resume chunks; expire abandoned multipart uploads.
- Blob uploaded but metadata commit fails: orphan cleanup/reconciliation.
- Metadata references missing blob: completion validates required objects before publishing version.
- Device retries same change: stable operation/version ID makes it idempotent.
- Share permission revoked: authorization checked at download token issuance; signed URL TTL should be bounded.
- Region outage: failover policy follows metadata RPO/RTO and blob-replication requirements.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Deduplication can reduce storage but increases privacy/security and indexing complexity. Strong global metadata writes are expensive; home-region ownership or active/passive failover may be simpler until global writes are required.

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

### Metadata operations should be small and strongly modeled

A file's identity, parent folder, owner, version pointer, permissions, and deletion state are metadata. These are tiny compared with file bytes but determine correctness. Keeping metadata in a transactional/queryable store makes operations such as rename, move, share, or commit-version easier to reason about.

### Upload sequence

```text
1. Client asks API to start upload.
2. API authenticates and allocates file/version/upload ID.
3. API returns signed chunk URLs or upload session.
4. Client uploads chunks directly to object storage.
5. Client reports completion with chunk hashes/manifest.
6. Service verifies required objects.
7. Metadata transaction publishes the new version.
8. Change event notifies other devices.
```

Until step 7, the incomplete upload should not appear as the current version.

### Chunking and deduplication

Content-defined or fixed-size chunks can reduce retransmission and storage. If chunks are globally deduplicated, encryption/privacy become harder because equal content maps to shared objects. Per-user dedupe is simpler from a privacy perspective but saves less space.

### Sync cursor

Each user/device consumes an ordered change stream. A cursor represents a durable position, for example server sequence number. If a device misses events while offline, it asks for changes after the cursor. Periodic full reconciliation catches rare missed/corrupt local state.

### Common interview mistakes

- Using the object store listing as the primary file-system metadata database.
- Publishing metadata before upload is complete.
- Assuming binary files can always auto-merge offline conflicts.
- Deleting chunks immediately without checking references/retention.
- Sending multi-GB files through app servers.
- Issuing long-lived signed URLs after permission revocation.

### Reusable patterns learned

Control plane vs data plane, multipart upload, immutable versions, change-log sync, conflict handling, tombstones, garbage collection, and signed object access.


## Detailed reference design

### Separate metadata from file bytes

This is the foundational decision.

```text
Metadata database
  file_id, owner, name/path, version, permissions, object manifest

Object storage
  immutable file chunks / versions
```

The database is optimized for listing, ownership, sharing, and sync. Object storage is optimized for durable large blobs. Do not make application servers proxy every multi-GB upload/download.

### File identity vs path

Treat `file_id` as stable identity and path/name as mutable metadata. Renaming `/docs/a.txt` to `/archive/a.txt` should not require copying the blob.

This also avoids path-based race problems and makes sharing/versioning easier.

### Upload flow

1. Client calls `init-upload` with file metadata/size/hash hints.
2. Service creates an `UPLOADING` version and returns signed multipart/chunk URLs.
3. Client uploads chunks directly to object storage, retrying failed chunks.
4. Client sends completion request with uploaded part IDs/checksums.
5. Service verifies parts and atomically publishes new version.
6. A change event notifies other devices.

The old version remains readable until the new one is committed, which prevents partial files from appearing.

### Chunking and deduplication

Fixed-size chunks are simple. Content-defined chunking can detect reused regions when a file changes but is more complex.

Chunks may be named by hash:

```text
sha256(content) → object key
```

This enables dedupe and integrity verification. Cross-user dedupe may leak information through timing/existence and should be evaluated carefully. Per-tenant/user dedupe is safer.

### Sync model

Every committed metadata change gets a monotonically ordered change sequence within the user's/account's scope.

Device stores:

```text
last_sync_cursor
```

On reconnect:

```text
GET /changes?after=cursor
```

The server returns create/update/delete/version events and next cursor. Periodic full reconciliation catches rare missed/corrupted local state.

### Offline edits and conflicts

Device A and B both edit version V1 offline.

A uploads V2 based on V1. B later uploads V3 also based on V1. The service detects B's base version is stale.

Possible policy:

- preserve both and mark conflict copy;
- automatically merge only for mergeable document formats;
- ask user to resolve.

Never silently overwrite without product-defined semantics.

### Deletion and garbage collection

Deletion usually creates a tombstone/metadata state first so all devices observe the deletion. Actual blob chunks are removed later when no live version references them and retention/undo policy permits.

Reference counting or mark-and-sweep can reclaim shared chunks.

### Sharing and authorization

Permissions:

```text
(file_id, principal_id, role)
```

Every metadata/download URL request checks permission. Signed download URLs are short-lived after authorization. Knowing a file ID or object key is not permission.

### Multi-region

A home-region model is reasonable: metadata writes for a user's account go to one region, while object replicas/CDN serve globally. This reduces conflict complexity. If data residency requires region-local storage, route objects/metadata by tenant policy.

## Failure walkthrough

### Upload completes in object store but API crashes before commit

Object is orphaned. A cleanup job deletes unreferenced upload parts after TTL. Retry completion is idempotent by upload/version ID.

### Metadata says version exists but object replication lags

Serve from source region or return temporary unavailable until object is present, depending on latency/residency requirements. Metadata publication can wait for minimum object durability if required.

### Sync event missed

Device cursor/reconciliation recovers from durable change log. Live push is an optimization, not source of truth.

### Rename races with delete

Use version/optimistic concurrency on metadata. One operation observes stale version and must retry/reconcile.

## Interviewer follow-ups

### “How do you download efficiently?”

Authorize via metadata API, issue signed CDN/object URL, and let bytes bypass app servers. Range requests support resume/streaming.

### “How do you support version history?”

File metadata points to current version, while Version rows reference immutable manifests/chunks. Retention policy determines how many old versions remain.

### “What if a file is shared with 1M people?”

Do not duplicate blob. Permission model can support groups/link shares; cache authorization metadata carefully. Download bytes scale via CDN.

### “What is the source of truth?”

Metadata DB for ownership/version state; object store for committed blobs. Client local copies and caches are derived.

## Common interview mistakes

- File path used as permanent identity.
- Large files proxied through app service.
- No resumable upload state.
- Sync based only on unreliable push events.
- Offline edits overwrite each other silently.
- Delete removes chunks immediately despite versions/shares.
- Object key treated as authorization.

## Short revision note

**File-sync pattern:** stable file ID + metadata DB + immutable object chunks/versions + signed direct transfer + change cursor + conflict detection + delayed GC.

## Topics to revise

- [ ] metadata vs blob
- [ ] file ID vs path
- [ ] multipart/chunk upload
- [ ] checksum/dedupe
- [ ] version manifest
- [ ] sync cursor/change log
- [ ] offline conflict
- [ ] tombstone/GC
- [ ] permissions/signed URL
- [ ] home-region strategy

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll separate stable file identity/metadata from immutable blob versions. Uploads go directly to object storage in chunks, metadata commit publishes a new version, and devices synchronize using a durable change cursor. Offline conflicts produce explicit versions rather than silent overwrite.

## How the design evolves at 10×

At 10× storage/users, object/CDN scales bytes independently. Partition metadata by user/tenant/home region, isolate huge shared files, and make change-stream/sync service horizontally scalable. GC/dedupe becomes a major background cost.

## Quick revision flashcards

**Why file ID not path?**  
Rename/move changes metadata without changing identity/blob.

**Sync source?**  
Durable metadata/change log, not push notification.

**Conflict?**  
Detect stale base version; preserve/merge according to file type.

**Delete?**  
Tombstone first; delayed GC after no live version references chunks.

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

**Next:** Lesson 35 — Design YouTube / Video Platform
