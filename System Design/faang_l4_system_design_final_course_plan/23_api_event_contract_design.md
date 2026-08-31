# Session 23 — API & Event Contract Design

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Design clear contracts that support pagination, retries, errors, evolution, and asynchronous behavior.

## What You Need to Read / Learn

- REST resource modeling and meaningful URLs.
- GET, POST, PUT, PATCH, DELETE semantics.
- Status codes and structured error responses.
- Cursor versus offset pagination.
- Filtering and sorting.
- API versioning/evolution.
- Idempotency keys for unsafe-to-repeat operations.
- Rate-limit headers/responses conceptually.
- gRPC/RPC versus REST at interview depth.
- Webhooks and retry/signature behavior.
- Event schema: event ID, type, version, entity ID, timestamp, payload; evolution compatibility.

## What You Need to Do

- [ ] Design APIs for messaging history, creating a post, and booking a seat.
- [ ] Design a cursor-pagination response for a feed.
- [ ] Design a webhook contract with event ID, signature, retry, and deduplication expectations.

## **Must Remember for the Interview**

- **API design is part of correctness: retry semantics and idempotency must be explicit.**
- **Cursor pagination is often better for changing/high-scale ordered feeds than deep offset pagination.**
- **HTTP method choice communicates expected semantics, but business idempotency still needs design.**
- **Events are contracts too; consumers need versioning/evolution rules.**
- **Do not expose internal database structure as the API by accident.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Define APIs after requirements and before detailed architecture.**
- **Include errors, pagination, retry/idempotency, and auth assumptions.**
- **Cursor → stable continuation token; offset → simple but expensive/unstable at deep pages.**
- **Webhooks need authentication/signing + retries + deduplication.**
- **Version contracts compatibly.**

## Self-Test Before Marking This Session Complete

- [ ] Can I design a REST API for a common system?
- [ ] Can I compare cursor and offset pagination?
- [ ] Can I explain idempotency at the API layer?
- [ ] Can I design an event/webhook contract?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 23/46  
**Next:** Session 24
