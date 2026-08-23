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


## Important interview ideas

> **Important:** Separate **large immutable bytes** from **small queryable metadata**. Then treat multi-region as a business requirement driven by latency, residency, and disaster recovery—not a default checkbox.

### Control plane vs data plane for files

For large upload/download:

```text
Control plane:
Client → API → metadata/auth → signed URL

Data plane:
Client ↔ object storage/CDN
```

This removes application servers from the high-bandwidth byte path while retaining authorization and metadata control.

### Multipart upload lifecycle

A robust upload can have states:

```text
INITIATED → PARTS_UPLOADING → COMMITTED → AVAILABLE
                         ↘ ABORTED/EXPIRED
```

The service records an upload ID and expected parts/checksums. Completion verifies required parts and atomically publishes a new file version. Background cleanup removes abandoned parts.

### Content addressing and dedupe

Chunks can be identified by cryptographic content hashes. Duplicate chunks may be reused. This can save storage but introduces privacy/security questions: never reveal whether another user's secret file exists based on dedupe timing/result. Cross-tenant dedupe may be inappropriate.

### Region strategies

**Home-region:** user/account writes go to one region; other regions serve cached/read replicas. Simpler conflict model.

**Active/passive:** secondary region waits for disaster. Simpler operations, higher failover delay and idle cost.

**Active/active:** multiple regions accept writes. Lower global write latency but conflicts and coordination are harder.

### RPO/RTO scenarios

If a photo service can lose 5 minutes of new uploads in a disaster and recover in 1 hour, asynchronous cross-region replication may be acceptable.

If a financial ledger requires RPO 0 and seconds of RTO, the architecture needs synchronous/quorum-like durability and much higher cost/complexity.

### Backups are not replicas

Replication copies corruption/deletion too. Backups provide historical recovery. Test restoration; an untested backup is only a hope.

## Worked scenario — global file service

Metadata is owned in the user's home region. Upload bytes go directly to regional object storage. Completed objects replicate asynchronously to a backup region. Downloads use CDN and can be served globally after replication.

For a regional outage, DNS/global routing moves control traffic to the DR region. RPO is determined by metadata/object replication lag. If that RPO is too large, synchronous metadata replication may be required even if large blobs remain asynchronous.

## Interview questions and model answers

### Q1. “Why presigned URLs?”

They let the app authorize an operation and give the client temporary scoped access to object storage. Large bytes avoid app-server bandwidth/CPU while authorization remains controlled.

### Q2. “Active-active or active-passive?”

I choose from write-latency, availability, RPO/RTO, conflict semantics, and cost. Active-active is not automatically better; for correctness-heavy data a home-region or active-passive model may be simpler.

### Q3. “Replica vs backup?”

A replica helps availability and recent durability but mirrors mistakes/corruption. A backup is a historical restore point, usually slower to recover from. Good systems may need both.

### Q4. “What dominates media cost?”

Stored bytes, transcoding/processing, and especially network egress/CDN delivery can dominate. I estimate bandwidth/egress rather than focusing only on API QPS.

## Common mistakes to avoid

- Proxying giant files through app servers.
- File blob stored in SQL row by default.
- Multi-region without consistency/RPO discussion.
- “Replication is backup.”
- No abandoned multipart cleanup.
- No checksum/integrity verification.
- Active-active chosen with no conflict model.

## Short revision note

Large-file pattern: **metadata DB + object store + direct signed transfer + chunk/checksum + versioning**. DR pattern: **failure domain → RPO → RTO → replication/backup/failover strategy**.

## Topics to revise

- [ ] object vs metadata
- [ ] presigned URL
- [ ] multipart/resume
- [ ] checksum/content hash
- [ ] multi-AZ vs multi-region
- [ ] home region / active-passive / active-active
- [ ] RPO/RTO
- [ ] backups vs replicas

## Interview-ready synthesis

### A strong 60–90 second explanation

I split large file metadata/control from blob transfer. Clients upload/download directly via signed URLs; multipart/chunks provide resume and checksums. For regions I choose home-region, active/passive, or active/active from latency, residency, RPO/RTO, and conflict requirements. Replicas and backups serve different recovery goals.

### How this topic connects to the wider system

- Performance: app servers avoid carrying large bytes; CDN serves global downloads.
- Reliability: multipart retry, replication, backup, and failover handle different failures.
- Correctness: metadata publishes only committed complete object versions.
- Cost: egress, replicas, storage tiers, and idle DR capacity dominate decisions.

### Revision flashcards with answers

**RPO?**  
Maximum acceptable data loss measured in time/state.

**RTO?**  
Maximum acceptable recovery time after failure.

**Replica vs backup?**  
Replica supports availability/recent copy; backup preserves historical restore points.

**Presigned URL?**  
Temporary scoped authorization for direct object-store operation.

**Active-active cost?**  
Conflict/coordination complexity and cross-region replication/egress.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Drive: metadata commits + object chunks; signed direct transfer handles large bytes.
- Video: object originals/renditions replicate and CDN delivers globally.
- DR: order metadata may need stronger RPO than reconstructable media thumbnails.

### When not to overuse this idea

Do not make every service active-active across regions. The operational/consistency cost must be justified by latency and availability requirements.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

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
