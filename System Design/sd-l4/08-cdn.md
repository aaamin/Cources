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
