# Lesson 25 — Object Storage, Media, Multi-Region & Disaster Recovery

**Phase:** Fundamentals  
**Session:** 25/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn object/blob storage, metadata separation, multipart uploads, regional topologies, failover, RPO/RTO, and cost awareness.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Object storage

Object stores hold large durable blobs—images, videos, documents, backups. Objects are addressed by keys and are not optimized for relational queries. Queryable metadata usually belongs in a database.

## Metadata vs blob data path

A file service often uses `API → metadata DB` while large bytes move `client ↔ object storage/CDN` using signed URLs. This keeps application servers out of the multi-gigabyte data path.

## Multipart/resumable upload

Split large files into independently retryable chunks, verify checksums, and commit a manifest after all parts exist. Abandoned multipart sessions need expiration/cleanup.

## Multi-AZ vs multi-region

Multiple availability zones protect from zone-level failure inside one region. Multiple regions protect from regional disaster and can reduce global latency, but cross-region consistency, routing, and cost become harder.

## Active/passive vs active/active

Active/passive keeps a standby region for failover and is simpler. Active/active serves writes/reads in several regions, improving locality but adding conflict and replication complexity.

## RPO and RTO

RPO asks how much recent data loss is acceptable. RTO asks how long service recovery may take. Near-zero RPO/RTO is expensive, so architecture should match business need.

## Cost awareness

Object retention, CDN egress, cross-region replication, and standby capacity can dominate cost. Resilience is requirement-driven, not a checkbox.

## Worked example — 10 GB file upload

The API creates metadata and returns pre-signed multipart upload URLs. The client uploads chunks directly, retries failures, and calls complete. The service verifies/checks the object and marks the version available. Regional replication follows the required RPO/RTO rather than automatically duplicating everything synchronously.

## Interview lens

For large media, separate metadata/control from blob delivery. For multi-region, state the failure/recovery objective before selecting active-active.

## What to remember

Object storage solves large durable blobs; multi-region design is driven by latency, residency, RPO/RTO, correctness, and cost.

## Review after reading

1. Why direct-to-object-store upload?
2. Why multipart?
3. Multi-AZ vs multi-region?
4. RPO vs RTO?
5. Why is active-active harder?

## Deeper study notes

### Blob immutability simplifies everything

Object storage works especially well when object versions are immutable. Updating a file can create a new object/version and atomically move metadata to point at it. Old versions can expire later. This avoids readers seeing a partially overwritten giant file.

### Direct upload changes trust boundaries

A pre-signed URL should be scoped to one object/key, method, size/content constraints where supported, and short lifetime. The client is authorized by your API, then transfers bytes directly. Completion still needs server-side validation before marking the file visible.

### Active-active writes create conflict questions

If two regions can update the same logical record during a partition, what happens? Last-writer-wins may lose valid changes; conflict-free merge is domain-specific; global consensus adds latency. A home-region strategy can avoid many conflicts by giving each record one write owner.

### Backups and replicas solve different failures

A replica faithfully copies accidental deletion/corruption. A backup/snapshot lets you recover historical state. Disaster recovery needs both redundancy and restore capability, plus actual restore testing.

### Common mistakes

- Calling a replica a backup.
- Routing huge uploads through API servers without reason.
- Choosing active-active only because the service is “global.”
- Ignoring egress/cross-region transfer cost.
- Claiming an RPO/RTO without architecture that can meet it.


## Personal notes

```text
Concepts that are clear:

Concepts to revisit:

Three things to remember:
1.
2.
3.

Questions for later:
```

---

**Next:** Lesson 26 — Observability, SLOs, Deployment, Security & Privacy
