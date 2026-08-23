# Lesson 08 — CDN & Edge Caching

**Phase:** Fundamentals  
**Session:** 8/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how CDNs reduce latency and origin load, what is cacheable, TTL/invalidation at the edge, signed URLs, and origin protection.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## What a CDN does

A CDN serves content from geographically distributed edges. `User → nearby edge → origin`. A cache hit avoids the origin, reducing user latency, backbone traffic, and backend load.

## Cacheable content

Images, video segments, JS/CSS, downloads, and public static assets are natural. Some dynamic responses can be cached when keys and freshness are safe. Highly personalized or rapidly changing responses are harder to share.

## Keys and TTL

The edge cache key often includes hostname/path and selected query/header dimensions. Bad keys either mix content incorrectly or destroy hit rate. Long TTL helps immutable assets; versioned URLs reduce invalidation pain.

## Origin protection

CDNs do not eliminate the origin. Mass cache expiry can send a miss storm upstream. Shield caches, request coalescing, immutable objects, staggered TTL, and sufficient origin capacity protect the source.

## Signed URLs

For private media, the application authorizes the user and issues a short-lived signed URL/cookie validated by the CDN. Large bytes then bypass application servers while preserving access control.

## Worked example — global product images

Store product images in object storage and expose them through a CDN. Use versioned immutable object names with long TTL. A product metadata update can reference a new image version rather than purging the old one globally.

## Interview lens

Whenever many users repeatedly download large/static content, ask whether a CDN should remove application servers from the byte path.

## What to remember

A CDN is geographically distributed caching/delivery. It improves latency and origin protection, especially for large immutable objects.

## Review after reading

1. Why CDN?
2. Why are versioned immutable assets easy?
3. What can overload origin despite CDN?
4. Why are personalized pages harder?
5. How do signed URLs help?

## Deeper study notes

### Cacheability is a data-property question

Ask whether a response can be shared between users, how often it changes, and how harmful staleness is. A versioned image is highly cacheable. A bank balance is not. A product catalog may be cacheable while inventory quantity needs fresher data. One page can therefore combine data from different freshness classes.

### Cache-Control and validation concepts

HTTP caching can use max-age/TTL, ETags, or last-modified validators. A stale edge can revalidate with the origin and receive “not modified” rather than downloading the whole object. You do not need header memorization, but recognize that expiration and validation are different strategies.

### Immutable naming is powerful

If content changes, publish a new key such as `app.8f3a.js` rather than modifying `app.js`. The old object can be cached for a long time because its bytes never change. The HTML points to the new version. This pattern dramatically reduces purge complexity.

### CDN is not a database

A CDN is optimized for delivery, not authoritative transactional state. It should usually serve copies whose loss or staleness has a defined fallback to an origin/source of truth.

### Common mistakes

- Routing large video bytes through application servers.
- Giving private objects long public cacheability without auth controls.
- Ignoring origin capacity because “we have a CDN.”
- Purging constantly changing assets instead of using versioned immutable URLs.


## Important interview ideas

> **Important:** A CDN is most valuable when the same bytes are requested by many users and those bytes can be served without consulting the origin for every request.

### Immutable content makes caching easy

If an image URL contains a content hash or version:

```text
/images/product123-v7.jpg
```

the object never changes at that URL. You can use a very long TTL. When the image changes, publish `v8`. This avoids complex global purge behavior.

Mutable URLs such as `/profile/alice.jpg` need shorter TTL, revalidation, or purge when content changes.

### Cacheability and privacy

A shared CDN cache must never accidentally serve private data to another user. Personalized responses may include authorization headers or cookies that make them unsuitable for shared caching unless the cache key and policy are designed carefully.

For private large files, a useful pattern is:

```text
Application authorizes request
        ↓
returns short-lived signed CDN/object URL
        ↓
client downloads bytes directly
```

The app controls permission without proxying the bytes.

### Cache miss storms

A CDN can still overload the origin if millions of users request an uncached object simultaneously. Techniques include:

- origin shield / second-level cache;
- request coalescing;
- pre-warming important content;
- immutable long-lived segments;
- enough origin capacity.

### Purge vs versioning

Global purge APIs are useful but can take time and add operational coupling. Versioned asset URLs are often safer for static resources because old cache entries naturally age out while new references point to fresh content.

## Worked scenario — live event video

For a live stream, the transcoder generates small media segments. The CDN caches each segment for viewers. Millions of viewers request the same recent segment, but only a small number of CDN shield/origin requests need to reach the storage/packager layer.

The system must ensure segments are uniquely versioned and origin can survive the first-request burst. Control APIs (chat, entitlement, metadata) scale separately from media delivery.

## Interview questions and model answers

### Q1. “CDN vs application cache?”

A CDN sits geographically near users and is excellent for HTTP-deliverable content, especially large/static bytes. An application cache is closer to backend services and stores objects/query results needed by business logic. A system can use both.

### Q2. “What is the origin?”

The authoritative upstream location the CDN fetches on a miss—often object storage, a media packager, or an HTTP service. The CDN is a delivery/cache layer; the origin remains the source for content generation/storage.

### Q3. “How do signed URLs work conceptually?”

The application authenticates/authorizes the user and signs a URL containing resource and expiration information. The CDN/object service verifies the signature before serving. This offloads bytes while keeping access time-bounded.

### Q4. “When is a CDN not useful?”

For highly personalized, rapidly changing, tiny responses with low reuse, edge hit rate may be poor and correctness complicated. The main benefit appears when content reuse and geographic distribution are substantial.

## Common mistakes to avoid

- Sending video through app servers unnecessarily.
- Assuming CDN means origin can never overload.
- Caching private content with unsafe keys.
- Using short TTL for immutable content.
- Forgetting invalidation/versioning strategy.
- Confusing CDN with durable source of truth.

## Short revision note

CDN = **edge delivery + caching**. Always think about cache key, TTL/versioning, origin behavior, privacy, and what happens on a miss.

## Topics to revise

- [ ] edge vs origin
- [ ] immutable/versioned assets
- [ ] TTL / purge
- [ ] signed URLs
- [ ] origin shield
- [ ] cacheability/privacy
- [ ] media delivery path
- [ ] CDN vs backend cache

## Interview-ready synthesis

### A strong 60–90 second explanation

I use a CDN when content is reused by many geographically distributed users and can be safely cached. I explain origin, cache key, TTL/versioning, private-content authorization, and cache-miss/origin protection. For immutable media, versioned URLs with long TTLs are usually simpler than frequent purge.

### How this topic connects to the wider system

- Performance: edge delivery reduces physical network distance and origin processing.
- Scalability: popular bytes are served many times without repeated origin requests.
- Security: signed URLs allow authorized direct delivery of private objects.
- Reliability/cost: origin shielding and high hit rate reduce egress/load spikes.

### Revision flashcards with answers

**What is origin?**  
The upstream source the CDN fetches on a miss, such as object storage or an HTTP service.

**Why version URLs?**  
Immutable URLs can be cached for long periods; new content gets a new URL.

**What is signed URL?**  
A time-limited authorization embedded in a URL/token validated by CDN/object storage.

**Can CDN fail origin?**  
A miss storm can; shield/request coalescing and origin capacity are still needed.

**CDN vs Redis?**  
CDN serves HTTP content near users; Redis-like cache serves backend/application data.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- YouTube: immutable media segments are ideal CDN objects with long TTL and high reuse.
- Dropbox: authorized downloads can use short-lived signed CDN/object URLs so bytes bypass app servers.
- News site: HTML/API may be dynamic while images/CSS are edge cached; caching policy can differ by content type.

### When not to overuse this idea

Do not CDN-cache sensitive personalized responses without a safe cache key and authorization design. Not everything at HTTP layer belongs in shared edge cache.

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

**Next:** Lesson 09 — Caching Fundamentals
