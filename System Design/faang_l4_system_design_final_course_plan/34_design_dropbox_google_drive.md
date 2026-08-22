# Session 34 — Design Dropbox / Google Drive

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice large objects, metadata, resumable transfer, sync, versions, conflicts, sharing, and cleanup.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: upload/download, folder metadata, sync, sharing, versions; decide offline editing scope.
- Metadata DB separate from blob/object storage.
- Multipart/resumable upload and checksums.
- Chunking/deduplication only if justified.
- Presigned URLs/direct object transfer.
- Change notification/sync cursor.
- Versioning and conflict detection.
- Offline edits and conflict resolution.
- Deletion/garbage collection.
- Regional residency as an extension.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Narrate a 10 GB resumable upload.
- [ ] Change request: two devices edit the same file offline.
- [ ] Describe deletion: metadata tombstone → eventual object garbage collection.

## **Must Remember for the Interview**

- **Separate file metadata from file bytes.**
- **Resumable uploads require upload IDs/parts/checksums and a commit step.**
- **Sync needs a version/change log so clients know what changed.**
- **Conflicts are product semantics; the system cannot magically merge arbitrary files.**
- **Deletion should prevent reads immediately even if physical object cleanup is asynchronous.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Client ↔ metadata service; large bytes ↔ object store directly.**
- **Use multipart/resumable transfer for large files.**
- **Track versions/change sequence for sync.**
- **Offline concurrent edits require conflict detection/resolution.**
- **Use tombstones/background GC for safe deletion.**

## Self-Test Before Marking This Session Complete

- [ ] Did I separate metadata and blobs?
- [ ] Did I design resumable upload?
- [ ] Did I define versioning/sync?
- [ ] Did I handle conflicts?
- [ ] Did I define deletion/GC?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support offline edits from multiple devices.

**Score using the 40-point rubric.**


---

**Progress:** Session 34/46  
**Next:** Session 35
