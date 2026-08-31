# Session 25 — Object Storage, Media, Multi-Region & DR

## 1. Must Learn

### Object storage & metadata separation
- **Understand:** Know large blobs belong in object storage while searchable/transactional metadata usually lives elsewhere.
- **Decision/trade-off:** Cheap scalable blob storage vs database-style querying/transactions.

### Large-object upload/delivery
- **Understand:** Understand multipart/resumable upload, checksums, presigned URLs, lifecycle/versioning, and CDN delivery.
- **Decision/trade-off:** Reliability/direct transfer vs control/security/complexity.

### Media processing pipeline
- **Understand:** Know upload → durable object → async processing/transcoding → metadata → CDN delivery.
- **Decision/trade-off:** Fast upload/request path vs eventual processing/retry needs.

### Single region, Multi-AZ, multi-region
- **Understand:** Distinguish fault scopes and why Multi-AZ is not regional disaster recovery.
- **Decision/trade-off:** Cost/complexity vs larger failure tolerance.

### Active/passive vs active/active
- **Understand:** Understand passive standby failover vs multiple serving/writing regions.
- **Decision/trade-off:** Simplicity/consistency vs global latency/availability and conflict complexity.

### Cross-region data strategy
- **Understand:** Know global routing, replication, home-region/ownership models, residency, and write-conflict issues.
- **Decision/trade-off:** Local latency/availability vs consistency/coordination.

### Disaster recovery
- **Understand:** Understand backup vs replica, RPO, RTO, restore testing, retention, and recovery validation.
- **Decision/trade-off:** Recovery objectives vs cost/operational complexity.

### Cost awareness
- **Understand:** Recognize storage, egress, replication, CDN, headroom, and active/active as major cost drivers.
- **Decision/trade-off:** Resilience/performance vs cost.

## 2. Should Know

- Origin protection for media delivery.
- Regional ownership/home-region is a practical way to reduce active/active write conflicts.
- Restore testing matters because an untested backup is not a proven recovery plan.

## 3. Recognition Only

- Erasure coding
- Multi-cloud DR
- Advanced global consensus

## 4. Important Comparisons

- Object storage vs database blob storage.
- Direct app upload vs presigned direct-to-object-storage upload.
- Single region vs Multi-AZ vs multi-region.
- Active/passive vs active/active.
- Replica vs backup.
- RPO vs RTO.
- Local writes vs globally coordinated writes.

## 5. Important Interview Questions

1. How are large files uploaded reliably without proxying all bytes through app servers?
2. What happens when processing/transcoding fails?
3. What happens if an entire region is unavailable?
4. How much data loss (RPO) and downtime (RTO) are acceptable?
5. Can two regions write the same logical data?
6. What data has residency constraints?
7. Which architecture costs dominate?

## 6. Common Interview Mistakes

- **Storing large media directly in relational DB by default** → Separate blob data from transactional metadata.
- **Proxying every large upload through app servers** → Consider presigned direct upload when appropriate.
- **Calling Multi-AZ multi-region DR** → They protect different failure scopes.
- **“Replica = backup”** → Replicas can copy corruption/deletion; backups support historical restore.
- **Active/active with no conflict model** → Define ownership or conflict resolution.
- **Defining RPO/RTO without restore testing** → Prove the recovery path.

## 7. Communication

### Important Vocabulary

object storage, blob, metadata, multipart upload, resumable upload, checksum, presigned URL, lifecycle policy, CDN, multi-AZ, active/passive, active/active, home region, RPO, RTO, backup, replica, failover

### Useful Interview Phrases

- “I’d keep blobs in object storage and transactional/searchable metadata in the database.”
- “Multi-AZ handles zone failure; regional disaster recovery is a separate requirement.”
- “A replica improves availability, but it is not a backup.”
- “Active/active reduces regional latency but makes cross-region writes and conflicts much harder.”

### Important Questions to Ask the Interviewer

- **Question:** “What are the largest object sizes and upload reliability needs?”  
  **Why it matters:** Determines multipart/direct upload design.
- **Question:** “What regional outage objective do we need?”  
  **Why it matters:** Determines single-region vs multi-region.
- **Question:** “What are the RPO and RTO targets?”  
  **Why it matters:** Determines backup/replication/failover investment.
- **Question:** “Can data be written from multiple regions?”  
  **Why it matters:** Determines conflict/ownership design.

## 8. ⭐ Must Remember

1. Store large blobs in object storage; keep metadata separately.
2. Use resumable/multipart upload for large unreliable transfers.
3. Process media asynchronously and make reprocessing safe.
4. Multi-AZ ≠ multi-region.
5. Active/active buys availability/latency at major consistency and cost complexity.
6. A replica is not a backup.
7. RPO = tolerable data loss; RTO = tolerable recovery time.
8. Test restores.

## 9. Study Priority

- **Priority A — Must master first:** object storage/metadata, upload/media pipeline, Multi-AZ vs multi-region, active/passive vs active/active.
- **Priority B — Must still learn:** cross-region ownership/replication/conflicts, RPO/RTO, backup vs replica, restore testing.
- **Priority C — Lower priority:** lifecycle/versioning details, fine-grained cost nuances, advanced global patterns.

## 10. Revision Checklist

- [ ] Design a large-object upload/media pipeline.
- [ ] Explain metadata vs object storage.
- [ ] Compare regional deployment models.
- [ ] Walk through total regional outage.
- [ ] Explain RPO/RTO and replica vs backup.
- [ ] Discuss conflict, residency, and main cost drivers.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
