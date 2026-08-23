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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 35 — Design YouTube / Video Platform
