# Session 35 — Design YouTube / Video Platform

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice asynchronous media processing, object storage, CDN delivery, transcoding, manifests, and origin protection.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: upload, process, watch; decide whether live streaming is initially in scope.
- Direct/resumable upload to object storage.
- Metadata service/database.
- Asynchronous processing/transcoding jobs.
- Multiple codecs/resolutions at conceptual depth.
- Manifest/adaptive bitrate concept.
- CDN delivery and cache behavior.
- Popular-video origin protection.
- Processing failure/retry and user-visible status.
- Moderation hooks at recognition depth.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design video-on-demand first.
- [ ] Change request: a live event attracts 20M viewers.
- [ ] Separate control plane, processing plane, and delivery plane on your diagram.

## **Must Remember for the Interview**

- **Large video bytes should flow through object storage/CDN, not normal app-server request paths.**
- **Upload and transcoding are asynchronous pipelines.**
- **Metadata and processing status need durable state.**
- **CDN cache hit rate/origin shielding dominate viral-content delivery capacity.**
- **Live streaming is a separate complexity jump; do not mix it into VOD without stating assumptions.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Upload → object store → processing queue → transcoders → renditions/manifests → CDN.**
- **Serving path should avoid transcoding/origin work per viewer.**
- **Adaptive bitrate lets clients select suitable rendition.**
- **Monitor processing backlog/failure and CDN/origin load.**
- **For massive live events, pre-position/scale distribution and protect origin.**

## Self-Test Before Marking This Session Complete

- [ ] Did I separate metadata, processing, and delivery?
- [ ] Did I use async transcoding?
- [ ] Did I explain CDN/origin behavior?
- [ ] Did I handle failed processing?
- [ ] Did I keep VOD and live requirements distinct?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Handle a massive live event traffic spike.

**Score using the 40-point rubric.**


---

**Progress:** Session 35/46  
**Next:** Session 36
