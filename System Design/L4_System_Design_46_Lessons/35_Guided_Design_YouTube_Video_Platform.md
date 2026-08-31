# Session 35 — Guided Design — YouTube / Video Platform

## Interview Prompt

> Design a YouTube-like video platform supporting upload, processing, playback, metadata, and popular videos.

Change request:
> A live event suddenly receives massive traffic.

Attempt for **40–50 minutes** first.

---

# STOP — Attempt First

Cover:
- upload;
- object storage;
- transcoding;
- manifests/renditions;
- CDN;
- metadata;
- async processing;
- viral content;
- origin protection;
- live-event adaptation.

---

# Reference Reasoning

## 1. Requirements

Core:
- upload video;
- process/transcode;
- playback;
- metadata/title/privacy;
- multiple qualities;
- huge read bandwidth.

Non-goals:
- recommendation ML;
- comments;
- ads;
- copyright internals unless asked.

Change:
- live streaming/highly popular event.

Non-functional:
- durable uploads;
- playback startup low latency;
- global availability;
- huge bandwidth;
- processing eventual;
- metadata consistency reasonable.

## 2. Scale

Assume:
- 1M uploads/day;
- average original 500 MB;
- massive playback.

Upload storage:
```text
~500 TB/day originals
```

Playback bandwidth dwarfs metadata APIs.

This immediately drives:
- object storage;
- CDN;
- direct upload;
- processing fleet.

## 3. Upload API

```http
POST /videos
→ video_id + signed multipart upload URLs
```

Client uploads directly to object store.

Finalize:
```http
POST /videos/{id}/complete
```

Video state:
```text
UPLOADING
PROCESSING
READY
FAILED
DELETED
```

## 4. Pipeline

```text
Client
  ↓ signed multipart
Object Store (original)
  ↓ event/outbox
Processing Queue
  ↓
Transcoding Workers
  ↓
Renditions: 360p/720p/1080p...
  ↓
Segment + Manifest Generation
  ↓
Object Store
  ↓
CDN
```

Metadata DB tracks processing state and object keys.

## 5. Why Transcode

Different:
- devices;
- bandwidth;
- screen size.

Generate multiple bitrates/resolutions.

Adaptive streaming:
- video split into small segments;
- manifest lists variants;
- client chooses bitrate based on network.

Recognition-level is enough.

## 6. Processing Queue

Transcoding is:
- CPU/GPU heavy;
- long-running;
- retryable.

Queue separates upload from processing.

Priority:
- short/new popular videos;
- premium/live content;
- normal backlog.

Monitor:
- queue age;
- processing duration;
- failure rate.

## 7. Idempotent Processing

Worker can retry.

Output paths versioned:
```text
video/{id}/version/{processing_version}/...
```

Metadata “current rendition set” switches atomically after successful complete.

Avoid partially replacing current video.

## 8. Playback Path

```text
Client → Video API → metadata + manifest URL
Client → CDN → segments
             └ miss → Object Store/Origin Shield
```

App server never carries all video bytes.

## 9. CDN

Popular video:
- edge hit rate high;
- origin protected;
- global latency low.

Use long TTL for immutable video segments.

New processing version uses new object paths, simplifying invalidation.

## 10. Viral Video

One video suddenly gets 100M views.

Risk:
- edge cold start;
- hot origin object;
- metadata hot key;
- comment/recommendation subsystems.

Mitigations:
- CDN;
- origin shield;
- prefetch/prewarm when known;
- local/cache metadata;
- rate/protect origin;
- multi-CDN optional at very high availability.

One hot video is precisely where CDN shines.

## 11. Upload Failure

Multipart upload:
- retry parts;
- resume;
- abandoned session cleaned later.

Finalize only when required parts/checksum valid.

## 12. Processing Failure

If rendition 1080p fails:
- retry;
- maybe publish lower qualities first if product allows;
- status shows processing.

Do not block all video availability unnecessarily if acceptable.

## 13. Metadata Store

Entities:
```text
Video
Uploader
Visibility
ProcessingVersion
Rendition
```

Reads:
- video by ID;
- user's videos;
- processing status.

SQL/document both possible.

Search index is derived later for title/content discovery.

## 14. Privacy

Private/unlisted:
- metadata authorization;
- signed playback URLs/tokens.

CDN cache can still serve encrypted/private bytes if access token mechanism works.

Deletion:
- mark metadata inaccessible quickly;
- purge/expire CDN;
- delete objects asynchronously according to retention.

## 15. Live Event Change Request

Live differs from VOD:

```text
Camera/Encoder
  ↓
Ingest endpoints
  ↓
Transcode/package live segments
  ↓
Origin
  ↓
CDN
  ↓
Millions of viewers
```

Critical concerns:
- continuous ingest;
- low segment latency;
- huge synchronized audience;
- origin protection;
- CDN capacity;
- failover encoder/ingest;
- live manifest updates.

### Massive traffic

Known event:
- pre-provision CDN/origin;
- pre-warm static manifests where possible;
- multiple ingest endpoints;
- multi-region origin;
- autoscale packaging;
- protect control APIs.

Viewer traffic should terminate mostly at CDN, not application servers.

## 16. Live Failure

If one origin region fails:
- CDN fetches from secondary origin;
- manifest/ingest failover.

If transcoder lags:
- buffer increases/playback latency;
- degrade quality;
- drop optional renditions.

Backpressure from ingest must be carefully handled because live cannot queue indefinitely.

## 17. Cost

Dominant:
- storage;
- transcoding compute;
- CDN/egress.

Optimize:
- lifecycle originals/old renditions;
- popular-content CDN hit rate;
- avoid unnecessary transcode combinations.

## Interview Questions

1. Why object storage?
2. Why direct upload?
3. Why queue transcoding?
4. What is adaptive streaming?
5. Why immutable segment paths?
6. What happens if CDN hit rate collapses?
7. How does live differ from VOD?
8. What breaks first at a giant live event?
9. How do private videos use CDN?

## Common Mistakes

- Video bytes via app servers.
- One giant file response rather than segment/CDN model.
- Transcoding synchronously in upload request.
- No processing state/idempotency.
- CDN with no origin protection.
- Live event treated identical to stored VOD.
- No bandwidth/cost reasoning.
- Private video publicly cacheable with no auth control.

## Must Remember

- **Object storage + CDN own the byte-heavy path.**
- **Metadata/API and media delivery are separate.**
- **Transcoding is asynchronous and idempotent.**
- **Adaptive streaming uses renditions/segments/manifests.**
- **Immutable outputs simplify caching/versioning.**
- **Viral/live viewer traffic should terminate at CDN.**
- **Live needs continuous ingest and cannot backlog indefinitely.**
- **Storage, compute, and egress dominate cost.**

## Self-Score

Use the 40-point rubric. If your diagram routes media through app servers, redraw the architecture from scratch.
