# Session 34 — Guided Design — Dropbox / Google Drive

## Interview Prompt

> Design a Dropbox/Google Drive-like file storage and synchronization system.

Change request:
> Two devices edit the same file while offline, then reconnect.

Attempt for **40–50 minutes** before reading.

---

# STOP — Attempt First

Cover:
- metadata vs content;
- upload/download;
- resumable/chunking;
- checksums;
- versioning;
- sync;
- offline edits;
- conflict resolution;
- sharing;
- deletion;
- consistency.

---

# Reference Reasoning

## 1. Requirements

Core:
- upload/download files;
- folders/metadata;
- multi-device sync;
- versioning;
- share file/folder;
- delete/restore maybe;
- large files;
- offline edits.

Non-functional:
- content durable;
- metadata strongly consistent enough for user operations;
- sync eventually completes;
- efficient bandwidth;
- large-object transfer should bypass app servers.

Clarify:
- collaborative live editing? Probably out of scope.
- max file size?
- dedup?
- consistency after upload?
- history retention?

## 2. Separate Metadata and Blobs

Metadata DB:

```text
File(
  file_id,
  owner_id,
  parent_id,
  name,
  current_version,
  size,
  status,
  updated_at
)

FileVersion(
  file_id,
  version,
  object_manifest,
  checksum,
  created_at,
  device_id
)
```

Object store:
- chunks/whole file data.

## 3. API

Create upload session:
```http
POST /files/{id}/upload-sessions
```

Upload chunks directly with signed URLs.

Finalize:
```http
POST /upload-sessions/{id}/complete
```

List changes:
```http
GET /sync/changes?cursor=...
```

Download:
- metadata API → signed CDN/object URL.

## 4. Upload Flow

```text
Client → Metadata/API: start upload
              ↓
        signed chunk URLs
Client → Object Storage: chunks
              ↓
Client → API: complete(manifest/checksums)
              ↓
metadata transaction creates new file version
              ↓
change event
```

Important:
**uploading bytes is not the same as committing the visible file version.**

Only after metadata commit does version become current.

## 5. Chunks

Large files can be split:
```text
chunk1 hash A
chunk2 hash B
...
manifest = [A,B,...]
```

Benefits:
- resume;
- retry only failed chunks;
- upload only changed chunks for some file types;
- dedup potential.

Costs:
- metadata complexity;
- chunk GC;
- security/privacy implications of global dedup.

At L4, chunking is enough; do not implement rsync algorithm unless asked.

## 6. Checksums

Client/server verify chunk/file integrity.

Checksum can also identify unchanged bytes.

But do not trust checksum alone for authorization.

## 7. Sync Model

Each account/device keeps a cursor/version.

Server maintains change log:

```text
seq 100: file A version 8
seq 101: folder B renamed
seq 102: file C deleted
```

Client:
1. sends last sync cursor;
2. gets changes after cursor;
3. downloads needed metadata/content;
4. advances cursor.

Push/WebSocket can notify “changes available,” but durable sync uses cursor/log.

## 8. Versioning

Use immutable file versions.

```text
file F current_version = 7
```

New upload creates version 8.

Advantages:
- rollback/history;
- conflict handling;
- cacheable immutable objects;
- atomic pointer switch.

Old versions retained according to policy.

## 9. Offline Conflict

Scenario:
- device A and B both sync version 5.
- both go offline.
- A uploads version 6A.
- B later uploads based on version 5.

Server must detect base-version mismatch.

Optimistic update:

```text
commit new version only if current_version == base_version
```

A wins:
```text
5 → 6
```

B submits base=5 while current=6:
- conflict.

Options:
1. create conflict copy (`report (B's conflicted copy).docx`);
2. merge automatically for mergeable formats;
3. ask user;
4. last-write-wins (risk data loss; weak for files).

For generic binary files, conflict copy is a strong default.

## 10. Directory/Filename Conflicts

Two devices create same filename in same folder.

Need uniqueness rule:
- allow duplicate names with IDs?
- enforce unique `(parent_id, normalized_name)` and resolve conflict.

IDs should be stable independent of path, so renames do not change identity.

## 11. Sharing

Metadata ACL:

```text
Share(file_id, principal_id/link_id, permission)
```

Access check before issuing download URL.

Public/share link:
- high entropy token;
- revocable;
- optional expiry/password.

If permission revoked, long-lived signed CDN URL can remain usable until expiry. Use short validity for sensitive files.

## 12. Deletion

Soft delete:
```text
status=TRASHED
```

Later permanent delete:
- remove metadata versions;
- decrement chunk references;
- object GC after safety delay;
- invalidate share links/search/cache.

Avoid deleting shared chunks still referenced by another version.

## 13. Dedup

Content-addressed chunks can be reused by hash.

Benefits:
- storage/bandwidth savings.

Risks:
- reference counting/GC;
- privacy side channels across users;
- encryption interaction.

Recognition depth only unless prompted.

## 14. Failure Scenarios

### Chunks upload, finalize fails
Chunks remain orphaned.
Cleanup by upload-session expiry/GC.

### Metadata commits, notification fails
Change log/outbox ensures sync event eventually published.

### Client finalizes twice
Upload-session completion idempotent.

### Object store temporarily unavailable
Metadata remains; download fails/degrades; retry/CDN cached versions may help.

## 15. Multi-Region

Assign account/file metadata home region for strong version ordering.

Blobs replicate globally/CDN.

User in another region may upload to nearby object store but metadata commit routes to home region.

Cross-region active metadata writes complicate conflicts; offline conflict logic already exists, but path/folder invariants still need authority.

## Interview Questions

1. Why separate blobs and metadata?
2. Why immutable versions?
3. How do offline conflicts get detected?
4. What if chunk upload succeeds but finalize fails?
5. How does sync recover after device is offline for a month?
6. Why stable file ID instead of path as identity?
7. What happens to chunks after deletion?
8. How does share revocation interact with signed URLs?

## Common Mistakes

- File bytes through API servers.
- “WebSocket sync” with no durable cursor.
- Last-write-wins losing offline edits.
- No base-version check.
- Path used as primary identity.
- Delete blob while another version references it.
- No orphan upload cleanup.
- No share authorization before signed URL.

## Must Remember

- **Metadata and bytes are different storage problems.**
- **Visible file version commits through metadata, not mere object upload.**
- **Immutable versions simplify sync/history/conflicts.**
- **Durable change cursor lets devices recover after offline periods.**
- **Offline conflicts use base-version optimistic check.**
- **Generic binary conflict copy is safer than blind last-write-wins.**
- **Object/chunk GC must respect references.**
- **Sharing authorization precedes direct object access.**

## Self-Score

Use the 40-point rubric. Redo offline-conflict and upload/finalize failure flows if correctness <3.
