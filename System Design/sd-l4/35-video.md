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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 36 — Design a Distributed Web Crawler
