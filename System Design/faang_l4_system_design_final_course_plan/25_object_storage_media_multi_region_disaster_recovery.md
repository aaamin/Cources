# Session 25 — Object Storage, Media, Multi-Region & Disaster Recovery

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand large-object storage/delivery and the main regional resilience choices without diving into vendor internals.

## What You Need to Read / Learn

- Object/blob storage and why large files should not usually pass through app servers.
- Metadata versus blob data.
- Multipart/resumable uploads and checksums.
- Presigned URLs.
- Versioning and lifecycle/tiering.
- Single region versus multi-AZ.
- Active/passive and active/active region strategies.
- Home-region/geographic ownership concepts.
- Cross-region replication and conflict risk.
- Global routing and failover.
- Backups versus replicas.
- RPO (acceptable data loss) and RTO (acceptable recovery time).
- Restore testing.
- Cost: storage, CDN/egress, replication, idle failover capacity.

## What You Need to Do

- [ ] Design upload/download of a 10 GB file using presigned URLs and multipart upload.
- [ ] Add a second region to a file service and state normal routing, failover trigger, and data behavior.
- [ ] Choose RPO/RTO for social photos versus payment records and explain the difference.

## **Must Remember for the Interview**

- **Store metadata and large blobs separately when their access/scaling patterns differ.**
- **Presigned/direct upload avoids proxying huge bytes through application servers.**
- **Multi-region adds latency, consistency, failover, and cost complexity; use it for a requirement.**
- **RPO is data-loss tolerance; RTO is recovery-time tolerance.**
- **A replica is not a backup; test restore procedures.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Object storage → durable large blobs; metadata DB → searchable/control data.**
- **Multipart/resumable upload handles large/unreliable transfers.**
- **Active/passive favors simpler consistency; active/active improves local availability but increases conflict/coordination complexity.**
- **State failover and failback behavior.**
- **Discuss RPO/RTO for regional disaster.**

## Self-Test Before Marking This Session Complete

- [ ] Can I design a large-file upload path?
- [ ] Can I explain presigned URLs?
- [ ] Can I compare active/passive vs active/active?
- [ ] Can I explain RPO vs RTO?
- [ ] Can I distinguish backup from replica?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 25/46  
**Next:** Session 26
