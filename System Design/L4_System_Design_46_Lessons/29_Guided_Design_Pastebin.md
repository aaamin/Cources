# Session 29 — Guided Design — Pastebin

## Interview Prompt

> Design Pastebin. Users create text pastes, receive a URL, read pastes later, and may choose expiration.

Change request:
> Support very large pastes and configurable expiration.

Attempt for **35–45 minutes** before continuing.

---

# STOP — Design It First

Include:
- API;
- ID;
- metadata/blob storage;
- cache;
- expiration;
- read path;
- abuse/security.

---

# Reference Reasoning

## 1. Requirements

Core:
- create paste;
- fetch by ID/alias;
- optional expiration;
- text may be public/unlisted/private if specified;
- delete may be optional.

Clarify:
- max paste size;
- edit allowed?
- custom aliases?
- syntax highlighting server/client?
- public discovery?

Non-functional:
- read-heavy;
- durable;
- low-latency reads;
- abuse controls;
- large objects should not overload DB/app.

## 2. API

Create:
```http
POST /pastes
{
  "content": "...",
  "expiresIn": 3600,
  "visibility": "unlisted"
}
```

For very large paste, prefer upload-session pattern:
```http
POST /pastes/uploads
→ signed upload URL
```

Read:
```http
GET /pastes/{id}
```

Metadata can return signed content URL for huge objects.

## 3. Data Model

For normal small text:

```text
Paste
-----
id
owner_id?
content or object_key
size
created_at
expires_at
visibility
status
hash/checksum?
```

Small payloads can live in DB/document store.

But requirement change says **very large pastes**. Separate:

```text
Metadata DB
Object Storage for content
```

This avoids giant rows and app bandwidth.

## 4. ID Generation

Same family as URL shortener:
- sequence + Base62;
- random ID with collision check;
- distributed ID.

Paste IDs should perhaps be hard to enumerate for unlisted privacy. Random high-entropy IDs are attractive.

Do not equate “unlisted” with strong access control. If truly private:
- authenticate/authorize;
- signed access.

## 5. Architecture

```text
Create small
Client → API → Metadata/Content DB

Create large
Client → API → signed upload → Object Store
                 ↓
              Metadata DB

Read
Client → API → Cache/Metadata
                 ↓
       content DB/object signed URL
```

Public content may use CDN.

## 6. Cache

Cache:
```text
paste-meta:{id}
paste-small:{id}
```

Immutable paste content is ideal for long TTL.

Expiration:
- cache TTL must not outlive paste logical access;
- or read path checks expiry.

If paste can edit, use versioned object/key or invalidate.

## 7. Expiration

Logical expiration:
```text
if now >= expires_at → unavailable
```

Physical deletion:
- background sweep;
- object lifecycle policy;
- partition/time index.

Do not synchronously scan/delete at exact second.

For large objects:
- metadata marks expiration;
- lifecycle/background deletes object.

Privacy/compliance deletion may have stronger timing requirements.

## 8. Very Large Paste Upload

Flow:

1. client asks create upload;
2. API validates user/quota/declared size;
3. generate paste ID + object key;
4. issue signed URL with size/type constraints;
5. client uploads multipart/resumable;
6. object store event/client completion;
7. server verifies object/checksum;
8. metadata becomes ACTIVE.

Use state:
```text
PENDING_UPLOAD
ACTIVE
EXPIRED
DELETED
```

Cleanup abandoned pending uploads.

## 9. Read Flow

Small:
```text
GET /id
 ↓
cache
 ↓ miss
DB
 ↓
return text
```

Large:
```text
GET /id
 ↓
authorize/check metadata
 ↓
signed CDN/object URL
 ↓
client downloads directly
```

App does not stream 500MB paste itself.

## 10. Abuse

Pastebin-like service is abuse-prone.

Controls:
- size limits;
- rate limits;
- malware/spam scanning for public content;
- content reporting/moderation;
- restrict executable rendering;
- HTML escape to prevent XSS;
- quotas;
- takedown workflow.

Never render arbitrary user HTML/script directly under trusted origin.

## 11. Privacy

Unlisted:
- secret-ish random URL;
- no index/listing.

Private:
- real auth/authorization.

If CDN caches private content:
- signed URL/cookie;
- correct cache key/permissions.

## 12. Failure Scenarios

### Object upload succeeds but metadata finalize fails
Have object key/upload ID; completion/reconciliation job can detect orphan/pending object.

### Metadata exists but object missing
Return processing/error; background repair/cleanup.

### Cache down
DB/object storage remains source.

### Cleanup worker down
Logical expiration still blocks reads; storage cleanup can lag.

This is why logical and physical expiration are separated.

## 13. Scaling

Read-heavy:
- cache metadata/content;
- CDN public immutable content.

Write:
- object storage scales bytes;
- metadata DB relatively small.

Partition metadata by paste ID if necessary.

Expiration cleanup:
- time buckets/lifecycle rather than full-table scans.

## Trade-Offs

| Choice | Benefit | Cost |
|---|---|---|
| Store text in DB | simple | giant rows for huge paste |
| Object storage | scales bytes | metadata/object dual state |
| Random ID | unguessable-ish | longer/collision logic |
| Sequence ID | compact | enumerable |
| Immutable paste | easy caching | no edit |
| Logical expiry first | fast correctness | physical bytes linger |

## Interview Questions

1. Why is Pastebin not identical to URL shortener?
2. When move content to object storage?
3. What if upload succeeds and DB write fails?
4. Difference unlisted vs private?
5. How do you expire 1B pastes efficiently?
6. How prevent XSS?
7. What if cache is down?

## Common Mistakes

- All content in DB regardless of size.
- App proxies giant files.
- Exact-time deletion scheduler per paste.
- Unlisted treated as authenticated private.
- No abuse limits.
- Rendering arbitrary text as HTML.
- Cache TTL beyond access expiration.
- No orphan-object cleanup.

## Must Remember

- **Metadata and large content should be separated.**
- **Immutable content is highly cacheable.**
- **Logical expiration can be immediate while physical deletion is async.**
- **Direct object upload/download avoids app bandwidth.**
- **Unlisted is not the same as private.**
- **Uploads need pending/finalized state and orphan cleanup.**
- **Abuse/security matter for user-generated text.**

## Self-Score

Apply the 40-point rubric, then redo the weakest 15 minutes.
