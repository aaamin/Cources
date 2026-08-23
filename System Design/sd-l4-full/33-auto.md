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


## Detailed reference design

### Separate build path from serving path

Autocomplete should not scan raw queries or documents on every keystroke. Build a compact **serving index** asynchronously.

```text
Query/search logs
   ↓
aggregation + cleaning
   ↓
ranking computation
   ↓
versioned suggestion index
   ↓
low-latency Suggest API
```

This is a reusable pattern: expensive computation offline/streaming, cheap lookup online.

### Serving data structure

For each prefix, store top K suggestions:

```text
"iph" → [("iphone", 98), ("iphone case", 91), ...]
```

A trie can represent prefixes compactly, but a distributed implementation may simply materialize prefix→topK in a key-value/search store. The interview does not require implementing a trie node by node unless asked.

### Build pipeline

Steps:

1. collect query signals;
2. normalize language/case/tokenization;
3. filter abuse/PII/unsafe suggestions;
4. aggregate frequency + recency;
5. calculate scores;
6. generate top K for prefixes;
7. publish a new immutable index version;
8. switch serving traffic after validation.

Versioned cutover allows rollback if ranking is bad.

### Ranking formula intuition

A simple score can combine:

```text
frequency
× recency decay
× quality/trust
× regional/language relevance
```

Exact ML is not needed. Explain inputs and offline/streaming refresh.

### Trending updates

Batch rebuild every hour may be too stale. Add a streaming “trend boost” layer that updates hot queries every minute, merged with the stable base index.

```text
stable global top-K
     +
recent trend deltas
```

### Personalization

Do not destroy cacheability for every request. Use a highly cached global/region prefix list and lightweight user reranking:

```text
global candidates
   ↓
optional user history rerank
```

Keep sensitive history bounded and privacy-aware.

### Partitioning

Partition by prefix hash or first characters. Alphabet ranges can skew heavily (`s`, `a`, etc.), so hash or dynamic splitting may be safer. Very short prefixes are the hottest and should be cached/replicated.

### API behavior

```http
GET /v1/suggest?q=iph&limit=10&locale=en-US
```

Debounce client keystrokes and require minimum prefix length to reduce QPS. The first 1–2 characters have enormous fan-out/hotness and low relevance.

## Failure walkthrough

### New index build fails

Keep serving previous immutable version. Build/cutover are separate. Never replace good index in place with partial data.

### Trend pipeline lag

Base suggestions still work; trends become less fresh. Monitor update lag rather than taking serving down.

### Bad suggestions released

Switch pointer back to prior version. Moderation/quality filters should run before cutover.

### Hot prefix

Edge/application cache top K for common prefixes. Replicate if one cache partition becomes hot.

## Interviewer follow-ups

### “Why not query Elasticsearch on every prefix?”

It can work at moderate scale, but precomputed top-K is cheaper and more predictable for extreme keystroke QPS. Search index may still be the build/source mechanism.

### “How do you avoid offensive suggestions?”

Moderation/quality filtering belongs in the build pipeline, with emergency denylist/override at serving if necessary. Do not expose raw user queries directly.

### “How often do you update?”

Match freshness requirement. Stable popularity can rebuild hourly/daily; trending layer updates in minutes/seconds. Separate base and delta avoids rebuilding everything continuously.

## Common interview mistakes

- Raw log scan on user request.
- Giant trie updated synchronously on every query.
- No index version/rollback.
- Ignoring hot short prefixes.
- Personalization that destroys all caching.
- No abuse/PII filtering.

## Short revision note

**Autocomplete pattern:** collect signals → precompute/version prefix top-K → cache hot prefixes → optional trend/personal rerank → safe cutover/rollback.

## Topics to revise

- [ ] prefix→topK
- [ ] trie concept
- [ ] offline/stream build
- [ ] ranking signals
- [ ] trending delta
- [ ] personalization vs cacheability
- [ ] hot prefixes
- [ ] versioned index cutover

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll model autocomplete as a precomputed serving problem: query/content signals are aggregated and ranked offline/streaming into a versioned prefix→top-K index. Online serving is a cache/key lookup, not a scan of raw logs.

## How the design evolves at 10×

At 10× keystrokes, debounce/min-prefix and cache hottest prefixes. At 10× corpus/signals, scale build pipeline and partition index. Keep versioned immutable cutover so rebuilds do not disrupt serving.

## Quick revision flashcards

**Why precompute?**  
Extreme read frequency demands predictable low-latency prefix lookup.

**Trie required?**  
No; conceptual prefix index/top-K store may be distributed more simply.

**Trending freshness?**  
Merge fast delta layer with stable base index.

**Rollback?**  
Serve versioned index and atomically switch pointer.

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

**Next:** Lesson 34 — Design Dropbox / Google Drive
