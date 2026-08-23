# Lesson 33 — Design Search Autocomplete

**Phase:** Guided Design  
**Session:** 33/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Return top suggestions for a typed prefix.
- Very low latency.
- Refresh as query trends change.
- Optionally personalize without breaking fallback/cacheability.

## 2. Scale and workload shape

Autocomplete can produce several requests per search because every keystroke may query. Estimate active search users × keystrokes/search and identify hot prefixes such as `a`, `i`, or popular brands. Serving is extremely read-heavy.

## 3. API / contract surface

```http
GET /v1/suggest?q=iph&limit=10
```

Keep responses small. Debounce on the client to reduce unnecessary traffic.

## 4. Data model

A serving representation can be:

```text
prefix → [(suggestion, score), ...]
```

The source signals may be query logs, catalog terms, editorial data, and abuse filters. The suggestion index is derived state.

## 5. High-level architecture

```text
Query Logs / Content
        ↓
Aggregation + Ranking
        ↓
 Versioned Suggestion Index
        ↑
Client → Suggest API → Cache
```

Serving is optimized for lookup; index building runs asynchronously.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Data structures

A trie explains prefix lookup conceptually. At scale, precomputing top-K per prefix or using a distributed search index may be easier to operate.

### Ranking

Combine frequency, recency, quality, language/region, and abuse filtering. Personalization can be a second-stage reranker rather than part of the globally cached base result.

### Index versioning

Build a new index version off-path, validate it, then atomically switch readers. Keep the old version for rollback.

## 7. Failure modes and recovery

- Index builder fails: continue serving previous version.
- Bad ranking rollout: roll back version.
- Hot prefix: cache locally/distributed/edge as appropriate.
- Personalization unavailable: serve global suggestions.
- Query-log abuse: filter spam/manipulated terms before publishing.
- Freshness lag: quantify acceptable update delay rather than forcing synchronous indexing.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

More precomputation reduces query latency but increases storage/update work. Personalization improves relevance but lowers cache sharing and increases privacy complexity.

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

### Build pipeline vs serving path

Autocomplete should have a cheap serving path and a heavier asynchronous build path. Do not recompute popularity/ranking for every keystroke.

```text
raw query/content signals
→ clean/filter
→ aggregate counts + recency
→ build ranked prefix data
→ validate/version
→ publish serving index
```

Serving then becomes a small lookup plus optional personalization.

### Prefix explosion

If every suggestion is stored under every prefix, storage multiplies by word length. That can still be acceptable because suggestions are short and top-K per prefix is small, but languages, long queries, and large vocabularies may require compression or search-index techniques.

### Client behavior matters

Debounce requests (for example after a short typing pause), do not query for every empty/one-character state unless useful, and cancel obsolete in-flight requests. Client behavior can reduce backend QPS by multiples.

### Personalization architecture

A useful pattern is:

```text
global cached top-K(prefix)
        ↓
optional user/region rerank
```

This preserves a highly cacheable base while allowing limited personalization. Avoid storing sensitive raw searches longer than required.

### Common interview mistakes

- Making the user request scan raw query logs.
- Updating one giant trie synchronously on every search.
- Ignoring ranking abuse/spam.
- Returning stale/bad index with no rollback version.
- Over-personalizing so every prefix lookup becomes uncacheable.

### Reusable patterns learned

Offline/stream precomputation, versioned derived indexes, top-K materialization, cacheable base + personalized rerank, and safe cutover/rollback.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 34 — Design Dropbox / Google Drive
