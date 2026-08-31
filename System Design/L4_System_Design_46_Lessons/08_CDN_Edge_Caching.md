# Session 8 — CDN & Edge Caching

## Outcome

You should be able to explain how a CDN works, which content belongs at the edge, how cache keys/TTL/invalidation affect correctness, how signed URLs protect private media, and how CDNs reduce latency, bandwidth, origin load, and cost.

## Mental Model

Without CDN:

```text
User in Dhaka
    ↓
Origin server far away
    ↓
Image/video response
```

With CDN:

```text
User
  ↓
Nearby Edge
  ├─ HIT → respond immediately
  └─ MISS → fetch from Origin → cache → respond
```

The CDN places cacheable content closer to users.

## Origin and Edge

### Origin
Authoritative backend/object store that owns content.

### Edge/Point of Presence
Distributed location near users that serves cached content.

The CDN is usually not the source of truth.

## Cache Hit and Miss

### Hit
Requested object is present and valid at edge.

Benefits:
- lower user latency;
- less origin bandwidth;
- less backend load.

### Miss
Edge retrieves from origin, may cache it, then returns it.

A sudden cold-cache event can create high origin traffic.

## What to Cache

Strong candidates:
- JavaScript/CSS;
- images;
- videos;
- public downloadable files;
- immutable versioned assets.

Possible with care:
- public API responses;
- HTML;
- personalized variants if cache key and privacy are correct.

Dangerous:
- private responses with insufficient cache key separation;
- frequently changing strongly consistent data;
- authorization-sensitive content accidentally cached publicly.

## Cache Keys

A cache key determines which requests share one cached object.

Typical dimensions:
- host;
- path;
- selected query parameters;
- sometimes headers.

Bad cache-key design can cause:
- low hit rate if irrelevant parameters create variants;
- privacy bugs if personalized content is shared.

Example:

```text
/avatar/user123?v=7
```

Versioned object names make immutable caching easier.

## TTL

Time-to-live controls how long cached content is considered fresh.

Long TTL:
- better hit rate;
- lower origin cost;
- slower update visibility unless versioned/purged.

Short TTL:
- fresher;
- more origin requests.

Use the product's staleness tolerance.

## Invalidation / Purge

When content changes before TTL expires:

Options:
- purge/invalidate edge object;
- use versioned URLs;
- use short TTL;
- stale-while-revalidate strategy.

Versioned immutable URLs are powerful:

```text
/app.abc123.js
```

New deployment uses a new URL instead of mutating the cached object.

## Signed URLs / Cookies Conceptually

Private media should not necessarily be world-readable because it lives on a CDN.

A backend can authorize the user and issue a time-limited signed URL/token.

```text
Client → App: may I access file X?
App → Client: signed URL valid for 5 min
Client → CDN/Object Store: GET signed URL
```

Benefits:
- application server stays off the media data path;
- access is time-bounded.

Be aware: once a client legitimately downloads content, cryptographic URLs cannot stop them from saving the bytes.

## Origin Protection

A CDN should prevent the origin from being directly overwhelmed/bypassed where possible.

Strategies:
- allow origin access only from CDN/network path;
- authenticated origin requests;
- rate protection;
- origin shield/mid-tier cache conceptually;
- autoscaling.

## Thundering Miss

If a very popular object expires simultaneously across many edges:

```text
Many misses → Origin spike
```

Possible mitigations:
- longer TTL;
- request coalescing;
- staggered refresh;
- stale serving;
- origin shield.

## CDN and Dynamic APIs

CDNs can cache more than static files, but dynamic caching requires careful semantics.

Example:
```http
GET /products/123
```
may tolerate 30 seconds of staleness.

```http
GET /account/balance
```
probably should not be shared cached.

Use explicit cache-control and authorization-aware design.

## Geo Benefit

A global user retrieving a 10 MB video from one distant origin repeatedly is inefficient. Edge delivery lowers:
- propagation latency;
- cross-region/origin bandwidth;
- origin connection count.

This is especially valuable for media.

## Cost Trade-Off

CDN itself costs money, but may reduce:
- origin egress;
- compute;
- bandwidth;
- scaling needs.

Whether it is cheaper depends on provider/pricing/workload. In interviews, identify the economic driver rather than quote vendor prices.

## Worked Example — Photo Sharing

Upload:
```text
Client → signed upload URL → Object Storage
App stores metadata
```

Read:
```text
Feed API returns metadata + CDN URL
Client → CDN
        ├─ HIT
        └─ MISS → Object Storage origin
```

For private photos:
- authorize access;
- signed URL/cookie;
- short validity;
- correct cache policy.

The app server does not proxy 5 MB images through itself.

## Small Design Drills

1. Why does versioning asset names make invalidation easier?
2. What happens if `user_id` is omitted from a cache key for a personalized page?
3. Why should large media usually be served via CDN/object storage rather than API servers?
4. What is an origin?
5. Why can a CDN outage or mass cache miss hurt the backend?
6. How can private content still use a CDN?

<details>
<summary>Answer key</summary>

1. New content uses a new URL; old immutable object may remain safely cached.
2. One user's personalized response could be served to another—a severe privacy bug.
3. Better bandwidth efficiency, edge latency, origin protection, and app-server resource use.
4. Authoritative source the edge fetches from.
5. Traffic that normally terminates at edges suddenly reaches origin.
6. Signed URLs/cookies plus correct authorization/cache policies.

</details>

## Common Mistakes

- Calling CDN the database/source of truth.
- Caching private content without correct key/auth semantics.
- Routing video bytes through application servers.
- Setting long TTLs on mutable URLs without invalidation.
- Assuming edge caches are always warm.
- Ignoring origin overload when cache hit ratio falls.
- Treating CDN as useful only for CSS/JS.

## Must Remember

- **CDN caches content near users.**
- **Origin remains authoritative.**
- **Cache key determines who shares cached content.**
- **TTL is a freshness-vs-hit-rate trade-off.**
- **Versioned immutable URLs avoid many invalidation problems.**
- **Signed URLs let private media bypass app servers safely after authorization.**
- **Mass misses can overload origin.**
- **CDN is especially valuable when bandwidth/media dominates.**

## Interview Revision Summary

Ask:

```text
What content is cacheable?
Public or private?
Cache key?
TTL?
How is change invalidated?
What happens on miss?
Can origin survive low hit rate?
Can client upload/download directly?
```

## Explain Without Notes

Design the media-delivery path for a private photo-sharing app. Include metadata API, object storage, CDN, signed URLs, and what happens after a photo is replaced.

## Completion Checklist

- [ ] I understand origin/edge/hit/miss.
- [ ] I can design cache keys/TTLs conceptually.
- [ ] I understand versioning and purge.
- [ ] I can use signed URLs for private media.
- [ ] I consider origin overload and cost.
