# Session 25 — Object Storage, Media, Multi-Region & Disaster Recovery

## Outcome

You should understand object storage and direct upload/download patterns, media processing/CDN flow, single-region vs multi-AZ vs multi-region architectures, active/passive vs active/active and home-region designs, cross-region conflicts/data residency, backups vs replicas, RPO/RTO, restore testing, and major cost trade-offs.

# Part A — Object Storage

## Why Object Storage

Large blobs such as:
- images;
- videos;
- documents;
- backups

are usually better in object storage than primary relational rows.

Object storage typically provides:
- durable blob storage;
- key/object namespace;
- scalable capacity;
- metadata;
- lifecycle/versioning;
- direct HTTP access controls.

Application DB stores metadata/reference:

```text
File(
  id,
  owner_id,
  object_key,
  size,
  content_type,
  checksum,
  created_at
)
```

## Metadata vs Object Data

Keep:
```text
permissions, owner, status, filename, object key
```
in metadata DB.

Keep:
```text
actual 2GB video bytes
```
in object store.

This separates transactional metadata from bandwidth-heavy storage.

## Presigned / Signed Upload

Instead of:

```text
Client → App Server → Object Store
```

use:

```text
1. Client → App: request upload
2. App authorizes and returns signed upload URL
3. Client → Object Store directly
4. Client/callback → App: complete
5. App verifies metadata/checksum/status
```

Benefits:
- app server avoids large-byte bandwidth;
- object store scales upload path.

Need security:
- limited expiry;
- object key restrictions;
- content-size/type validation;
- final verification.

## Multipart / Resumable Upload

Large file is split into parts.

Benefits:
- retry only failed part;
- parallel upload;
- resume interrupted transfer.

Need completion/finalization semantics.

## Checksums

Checksum verifies uploaded bytes match expected content.

Useful for:
- corruption detection;
- dedup recognition;
- resumable integrity.

Checksum is not user authentication.

## Lifecycle Policies

Move/delete data by age:

```text
hot storage → cold/archive → deletion
```

Useful for:
- logs;
- old backups;
- media versions.

Align with retention/privacy requirements.

## Versioning

Keep old object versions to protect against accidental overwrite/delete.

Costs storage.

Versioning does not replace independent backup policy by itself.

# Part B — Media Pipeline

Example video:

```text
Upload
  ↓
Object Storage (original)
  ↓ event
Transcoding Queue
  ↓
Workers
  ↓
Multiple renditions + manifest
  ↓
Object Storage
  ↓
CDN
  ↓
Viewer
```

Metadata DB tracks:
- upload status;
- processing status;
- renditions;
- permissions.

## Processing Is Async

Video transcoding may take minutes.

API returns:
```text
UPLOAD_ACCEPTED / PROCESSING
```

Worker retries failures.

Do not keep client HTTP request open for a 20-minute transcode.

# Part C — Availability Domains

## Single Instance

One machine failure = outage.

## Multi-AZ / Multiple Failure Zones

```text
Regional LB
  ├─ AZ A app
  └─ AZ B app
        ↓
Multi-AZ data layer
```

Protects against instance/AZ failures while keeping low regional latency.

Often sufficient for many systems.

## Multi-Region

Use when requirements justify:
- regional disaster resilience;
- global latency;
- data residency;
- massive geography;
- business continuity.

Multi-region is expensive/complex. Do not add it to every interview.

# Part D — Active/Passive

```text
Primary Region → serves traffic
Secondary Region → replicated/standby
```

On disaster:
- promote secondary;
- route traffic.

Advantages:
- simpler write ownership;
- lower cost than full active/active.

Costs:
- failover time;
- standby may be stale;
- need test;
- capacity ramp.

# Part E — Active/Active

Multiple regions serve traffic.

Can mean:
- active reads everywhere, writes one region;
- local writes in each region;
- partition users by home region.

Do not say “active-active” without defining write model.

## Home-Region Model

Assign each user/entity a home region:

```text
user A → Singapore
user B → Frankfurt
```

Writes go to home region.
Other regions may proxy/cache/replicate.

Benefits:
- avoids many concurrent cross-region conflicts;
- local experience for home users;
- clear ownership.

Challenges:
- user travel;
- cross-region interactions;
- failover home ownership.

## Cross-Region Write Conflicts

If same entity writable in two regions during partition:

```text
Region A writes name=X
Region B writes name=Y
```

Need:
- conflict resolution;
- single/home writer;
- global consensus;
- merge policy.

For correctness-critical inventory/money, often prefer ownership/coordination rather than arbitrary last-write-wins.

## Data Residency

Some data may need to remain in region/country.

Architecture implications:
- home-region placement;
- limited replication;
- separate keys/storage;
- routing based on tenant residency.

Do not claim legal requirements you do not know; state it as a product/compliance constraint to confirm.

# Part F — Disaster Recovery

## Backup vs Replica

Replica:
- current copy for serving/failover;
- reproduces accidental deletion/corruption.

Backup:
- recovery point from past.

Need both for different failures.

## RPO — Recovery Point Objective

Maximum acceptable data loss in time.

Example:
```text
RPO = 5 minutes
```
In disaster, business accepts at most ~5 minutes of lost changes.

RPO=0 requires much stronger synchronous/durable replication.

## RTO — Recovery Time Objective

Maximum acceptable recovery time.

Example:
```text
RTO = 30 minutes
```

Determines:
- warm standby;
- automation;
- restore strategy;
- capacity readiness.

## Restore Testing

A backup that has never been restored is an assumption.

Test:
- can files be read?
- can DB restore?
- how long?
- are encryption keys available?
- are dependencies/configurations captured?
- can DNS/routing failover?

## Regional Outage Plan

Ask:
1. Detect outage.
2. Stop unsafe writes / decide authority.
3. Promote secondary/home reassignment.
4. Route traffic.
5. Validate data.
6. Recover old region.
7. Reconcile conflicts/missing events.
8. Fail back carefully.

# Part G — Cost

Major drivers:
- compute;
- storage;
- replicas;
- cross-region transfer;
- internet egress;
- CDN;
- backup retention;
- active-active duplicated capacity.

A design with 3 active regions may roughly multiply several infrastructure costs and operational burden.

Cost is a non-functional requirement, not an afterthought.

## Worked Example — Global Photo App

Base:
```text
Users → Regional API
Metadata DB in home region
Images → Object Storage
Images served → CDN
```

Upload:
- signed URL to nearest/appropriate region;
- metadata commit;
- image processing async.

DR:
- metadata replicated to standby region;
- objects cross-region replicated according to RPO/residency;
- active/passive initially.

At larger global scale:
- users assigned home region;
- CDN handles read bandwidth globally;
- avoid active/active metadata writes unless justified.

## Small Design Drills

1. Why use presigned uploads?
2. What is the difference between RPO and RTO?
3. Why isn't a replica a backup?
4. Why can active-active writes create conflicts?
5. When might home-region ownership help?
6. Why is CDN more important for video than metadata?
7. What does restore testing prove?

<details>
<summary>Answer key</summary>

1. Client transfers large bytes directly to storage, reducing app-server bandwidth while preserving authorized limited access.
2. RPO = acceptable data loss; RTO = acceptable recovery duration.
3. Replicated logical errors/deletes can destroy all replicas; backup is historical recovery.
4. Same entity can be changed independently during delay/partition.
5. Clear single write authority/locality while still supporting global routing.
6. Media bandwidth is huge and benefits from edge proximity/origin offload.
7. Backup is usable and recovery procedure meets timing/dependency requirements.

</details>

## Common Interview Mistakes

- Storing giant video blobs in relational DB without reason.
- Proxying all uploads/downloads through app servers.
- Adding multi-region because “high availability” with no requirement.
- Saying active-active without write/conflict model.
- Confusing multi-AZ with multi-region.
- Replica = backup.
- Mentioning backups without restore tests.
- RPO/RTO reversed.
- Ignoring egress/cross-region cost.
- Ignoring data residency.

## Must Remember

- **Large blobs belong in object storage; transactional metadata elsewhere.**
- **Signed URLs keep app servers off the large-byte path.**
- **Media processing is naturally asynchronous.**
- **Multi-AZ handles zone failures; multi-region handles larger geography/disaster requirements.**
- **Active-active must define write ownership/conflicts.**
- **Home-region ownership can simplify global writes.**
- **RPO = acceptable data loss; RTO = acceptable recovery time.**
- **Replica is not backup.**
- **Restore tests are mandatory for real DR confidence.**
- **Cross-region and egress costs can dominate.**

## Interview Revision Summary

Object/media:
```text
metadata?
object store?
signed upload/download?
multipart?
checksum?
processing queue?
CDN?
```

Region/DR:
```text
single/Multi-AZ/multi-region?
why?
active/passive or active/active?
write owner?
replication?
conflicts?
residency?
RPO?
RTO?
backup?
restore test?
cost?
```

## Explain Without Notes

Design upload, processing, delivery, and DR for a global video platform. Explicitly distinguish metadata storage, object storage, CDN, multi-AZ, multi-region, RPO, and RTO.

## Completion Checklist

- [ ] I understand object-storage patterns.
- [ ] I can design direct/resumable upload.
- [ ] I understand media processing/CDN.
- [ ] I compare multi-AZ/multi-region.
- [ ] I define active/passive/active-active writes.
- [ ] I understand RPO/RTO/backups/restores.
- [ ] I consider residency and cost.
