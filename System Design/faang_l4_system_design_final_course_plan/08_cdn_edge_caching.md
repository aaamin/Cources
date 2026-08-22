# Session 08 — CDN & Edge Caching

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn how content is moved closer to users and how edge caching changes latency, bandwidth, and origin load.

## What You Need to Read / Learn

- CDN edge locations and origin servers.
- Cacheability: static assets, images, video segments, and selected dynamic content.
- Cache keys and variants.
- TTL and freshness.
- Cache purge/invalidation.
- Cache miss and origin fetch behavior.
- Signed URLs/cookies conceptually for private content.
- Origin protection and shielding.
- Regional latency and bandwidth/egress implications.

## What You Need to Do

- [ ] Design the delivery path for a public image and a private video.
- [ ] Explain how a stale asset is replaced after deployment.
- [ ] Describe what happens during a viral traffic spike when the object is already cached versus not cached.

## **Must Remember for the Interview**

- **A CDN reduces latency and origin load only for content that can be served from the edge.**
- **Cache key design determines whether users receive the correct variant.**
- **Long TTL improves cache efficiency but increases staleness risk.**
- **Signed URLs can authorize direct object/CDN access without proxying large bytes through app servers.**
- **Protect the origin from cache-miss storms.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **CDN = geographically distributed edge cache/delivery layer.**
- **Use for static assets and large media whenever appropriate.**
- **Think about TTL, invalidation, private access, and origin failure.**
- **Do not route large media through application servers without a reason.**
- **A CDN is often a latency + bandwidth + origin-capacity optimization.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain CDN vs application cache?
- [ ] Can I describe a cache miss?
- [ ] Can I explain signed URLs?
- [ ] Can I discuss TTL/freshness trade-offs?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 8/46  
**Next:** Session 9
