# Lesson 35 — Design YouTube / Video Platform

**Phase:** Guided Design  
**Session:** 35/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Upload videos.
- Transcode to multiple codecs/resolutions.
- Stream globally with low startup latency.
- Keep media delivery separate from metadata APIs.

## 2. Scale and workload shape

Estimate uploads/day, average upload size, views/day, average watch duration/bitrate, concurrent viewers, and egress. For video, **delivery bandwidth** often dominates normal API QPS.

## 3. API / contract surface

```http
POST /v1/videos/init-upload
POST /v1/videos/{id}/complete
GET  /v1/videos/{id}
```

Playback metadata returns a manifest or CDN URLs. App servers should not proxy the video stream itself.

## 4. Data model

```text
Video(video_id, owner_id, title, status, manifest_ref, ...)
TranscodeJob(video_id, profile, state, attempts)
```

Originals, encoded segments, thumbnails, and manifests live in object storage/CDN.

## 5. High-level architecture

```text
Uploader → Object Storage
             ↓
         Processing Queue
             ↓
      Transcode Workers
             ↓
    Encoded Object Storage
             ↓
            CDN
             ↑
          Viewers

Metadata API → Metadata DB
```

Upload and processing are asynchronous; playback becomes available when policy-required outputs are ready.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Transcoding

Create multiple jobs for resolution/codec/audio profiles. Jobs write deterministic outputs so retries are safe. A workflow marks video READY when enough required variants succeed.

### Adaptive bitrate

The player downloads a manifest describing segment variants and switches bitrate based on bandwidth/buffer. This moves adaptation to the client and CDN-friendly segment layer.

### CDN and origin protection

Encoded segments are immutable and ideal for long-lived caching. Shield layers/request coalescing protect object-storage origin on cache misses.

### Live events

Live video replaces an offline pipeline with continuous segment generation. Capacity planning centers on ingestion + CDN fan-out; metadata traffic is tiny in comparison.

## 7. Failure modes and recovery

- Transcoder crashes: retry idempotently; DLQ persistent bad inputs.
- One rendition fails: decide minimum set required for READY.
- CDN cache miss storm: origin shield, long TTL, pre-warm known event assets.
- Object storage unavailable: uploads/processing pause; already cached media may still stream.
- Metadata service down: CDN URLs already held by active players may continue temporarily.
- Copyright/moderation pipeline delayed: define whether publication waits or uses post-publication review.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

More renditions improve device/network quality but increase compute/storage. Media bytes belong in object/CDN paths; app servers control metadata/auth, not bulk delivery.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Upload is control-plane plus data-plane

The API should allocate video ID and upload permissions; the bytes go directly to object storage. After completion, the original object is immutable input to a processing DAG. This means users can retry metadata calls without re-uploading gigabytes.

### Processing DAG

Video processing is rarely one job. A pipeline may include validation, malware scan, metadata extraction, thumbnail generation, several video encodes, audio tracks, subtitles, moderation, and manifest generation. Represent jobs durably so failed stages retry independently.

### Segmenting makes CDN delivery efficient

Instead of one huge video file, package into short segments. The player requests segments listed by a manifest and can switch quality at segment boundaries. Segments are immutable and cache extremely well at CDNs.

### Popularity distribution

Video views are highly skewed. Most videos may receive few views while a small set drives enormous bandwidth. CDN caching naturally handles this: popular segments remain at the edge; unpopular content falls back to origin/object storage.

### Live-event difference

For live streaming, segments are produced continuously. The system cannot precompute every rendition hours ahead. Ingest, transcode, package, and distribute operate as a low-latency pipeline. The CDN still provides fan-out, while origin shields protect the live packager from millions of viewers.

### Common interview mistakes

- Proxying uploads/playback through application servers.
- Making upload request wait for full transcoding.
- Storing media blobs in relational rows.
- Forgetting idempotent transcode output/retry.
- Treating CDN as optional for tens of millions of viewers.
- Mixing metadata database scale with video egress scale.

### Reusable patterns learned

Asynchronous DAG processing, object storage, immutable segments, manifests, adaptive bitrate, CDN fan-out, origin protection, and retryable media pipelines.


## Detailed reference design

### Split the system into three planes

A useful interview model:

1. **control/metadata plane** — video records, titles, permissions, status;
2. **processing plane** — upload, transcoding, thumbnails, moderation;
3. **delivery plane** — manifests, media segments, CDN.

These planes have very different scale. Playback bytes dominate bandwidth; metadata QPS is comparatively small.

### Upload flow

```text
Creator → Video API → create video_id / upload session
Creator → signed URL → object storage (original)
Creator → complete
                  ↓
            VideoUploaded event
                  ↓
            processing DAG
```

Direct upload prevents application servers from carrying gigabytes.

### Processing DAG

A video may need:

- virus/format validation;
- transcoding 360p/720p/1080p;
- audio extraction;
- thumbnails;
- subtitles/moderation;
- packaging segments/manifests.

Represent tasks as durable jobs. Individual renditions retry independently. Outputs use deterministic object keys so retries are idempotent.

Video state can evolve:

```text
UPLOADING → PROCESSING → PARTIALLY_READY → READY / FAILED
```

A product may publish lower resolution first while high resolution finishes later.

### Adaptive bitrate streaming

Instead of one giant file, store media as small segments at several bitrates. A manifest describes available variants. The player measures bandwidth/buffer and switches variants segment by segment.

This improves playback under changing network conditions.

### CDN path

```text
Viewer
  ↓
CDN edge
  ↓ miss
origin shield/CDN layer
  ↓
object storage / packager
```

Media segments are immutable and ideal for long cache TTLs. CDN hit rate and egress dominate cost/performance.

### Origin protection

Popular content can cause many edge locations to miss at once. Use shield caches, request coalescing, pre-warming for scheduled live events, and scalable origin bandwidth.

### Live streaming extension

Live ingest differs from uploaded video:

```text
Live encoder → regional ingest → realtime transcode/package → short segments → CDN
```

Latency target determines segment size and buffering. Millions of viewers should still consume through CDN; origin should not maintain one stream per viewer.

### Metadata/search/comments

Keep these out of the media byte path. Metadata service can use relational/document storage; search uses a derived search index; comments are a separate realtime/social system.

### View counting

Do not synchronously increment one database row for every view. Emit view events to a stream and aggregate. Displayed counts can be eventually consistent.

## Failure walkthrough

### Transcoder worker crashes

Job lease expires and another worker retries. Deterministic output paths prevent duplicate renditions.

### One rendition fails

Keep other renditions available. Retry failed variant. Depending on product, mark video READY when minimum set exists.

### CDN outage/poor hit rate

Traffic can overload origin. Multi-CDN or fallback is possible at large scale; at minimum have shield/origin capacity and monitoring.

### Metadata unavailable

Cached manifests/media may continue playing for existing sessions, but new discovery/auth requests fail. This shows plane isolation.

## Interviewer follow-ups

### “How do you handle a 20M-viewer live event?”

Scale ingest/transcoding enough to create one set of media segments, then CDN fans those segments to viewers. Pre-warm/shield caches, reserve capacity, and monitor origin requests/egress. Viewer count should not linearly load the transcoder.

### “Why multiple resolutions?”

Different devices/network conditions. Adaptive bitrate reduces buffering and avoids sending 4K bytes to a weak connection.

### “How do you estimate storage?”

Original size + sum of encoded renditions × retention. Transcoded copies can multiply storage, but delivery egress may cost even more for popular content.

### “When is video visible?”

Define state. Could show PROCESSING, publish once minimum rendition is ready, and progressively add higher quality.

## Common interview mistakes

- Stream video through app servers.
- One synchronous upload request performs transcoding.
- No durable processing job state.
- Viewer traffic sent directly to object origin with no CDN.
- View counter as one hot row.
- Metadata/search/comments mixed into media pipeline.
- “CDN handles everything” with no origin-miss story.

## Short revision note

**Video pattern:** direct upload → durable processing DAG → immutable segmented renditions/manifests → CDN delivery. Separate metadata, processing, and delivery planes.

## Topics to revise

- [ ] direct upload
- [ ] processing DAG
- [ ] idempotent transcoding jobs
- [ ] ABR segments/manifest
- [ ] CDN/origin shield
- [ ] live ingest extension
- [ ] async view aggregation
- [ ] cost: storage + egress

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll separate metadata/control, asynchronous processing, and CDN delivery. Creators upload directly to object storage; a durable DAG transcodes into immutable renditions/segments; viewers fetch manifests/segments through CDN. Media bandwidth, not metadata QPS, is the dominant scale.

## How the design evolves at 10×

At 10× viewers, CDN and origin shielding scale; transcoding does not scale with views. At 10× uploads, increase processing workers/storage. For live events, reserve ingest/transcode capacity and prewarm/shield delivery.

## Quick revision flashcards

**Why segments?**  
Adaptive bitrate and cacheable small immutable media units.

**Transcode retry?**  
Deterministic output keys/idempotent jobs.

**View count?**  
Async event aggregation, not one hot DB row.

**Origin protection?**  
CDN shield/coalescing/prewarm plus capacity.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 36 — Design a Distributed Web Crawler
