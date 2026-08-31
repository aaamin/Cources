# Session 29 — Design Pastebin

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice IDs, expiration, read-heavy storage, large payload handling, and a deliberately simple architecture.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: create paste, read paste, expiration, optional private/unlisted behavior.
- Scale: paste creation rate, read ratio, average/maximum paste size, retention.
- API/data: paste ID, content/object reference, owner, created_at, expires_at.
- Storage choice: DB for metadata/content when small; object storage for large blobs.
- ID generation and collision behavior.
- Cache popular paste reads.
- Expiration cleanup: lazy expiration plus background cleanup.
- Abuse: payload limits/rate limits.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design a minimal version first; do not begin with microservices.
- [ ] Change request: support very large pastes and configurable expiration.
- [ ] Compare storing content in DB versus object storage.

## **Must Remember for the Interview**

- **Pastebin should stay simple; complexity must be justified by payload size or traffic.**
- **Expiration has read-time correctness and background-cleanup aspects.**
- **Large blobs can move to object storage while metadata stays searchable in a DB.**
- **IDs need uniqueness, not necessarily global ordering.**
- **Cache only if read traffic justifies it.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Simple flow: create → generate ID → store → read by ID.**
- **For large content: metadata DB + object store + optional direct transfer.**
- **Expired objects should not be returned even before physical cleanup.**
- **State max size/retention assumptions.**
- **Avoid premature distributed complexity.**

## Self-Test Before Marking This Session Complete

- [ ] Did I keep the first version simple?
- [ ] Did I state maximum paste size and retention?
- [ ] Did I define expiration semantics?
- [ ] Did I choose DB vs object storage based on payloads?
- [ ] Did I explain ID generation?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support large pastes and configurable expiration.

**Score using the 40-point rubric.**


---

**Progress:** Session 29/46  
**Next:** Session 30
