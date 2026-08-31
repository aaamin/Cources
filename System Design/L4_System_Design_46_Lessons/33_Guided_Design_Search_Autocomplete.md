# Session 33 — Guided Design — Search Autocomplete

## Interview Prompt

> Design search autocomplete. As users type a prefix, return the top suggestions quickly.

Change request:
> Support trending queries and personalized suggestions.

Attempt for **35–45 minutes** before continuing.

---

# STOP — Attempt First

Cover:
- query prefix access;
- ranking;
- data collection;
- index structure;
- cache;
- refresh pipeline;
- freshness;
- trending;
- personalization;
- abuse/privacy.

---

# Reference Reasoning

## 1. Requirements

Core:
- input prefix;
- return top K suggestions;
- low latency (<100ms-style target);
- suggestions based on popular historical searches;
- update periodically.

Change:
- trending near realtime;
- personalization.

Clarify:
- languages?
- typo tolerance?
- adult/unsafe terms?
- geographic ranking?
- scale of vocabulary?
- how fresh?

Non-goals:
- full search result ranking;
- web crawling.

## 2. API

```http
GET /autocomplete?q=app&limit=10
```

Response:
```json
[
  {"text":"apple", "score":...},
  {"text":"apple store", "score":...}
]
```

Optional context:
- locale;
- region;
- authenticated user.

Do not expose sensitive raw personalization context unnecessarily.

## 3. Data Sources

Search/log stream:

```text
SearchExecuted(query, user?, region, timestamp)
```

Pipeline:
- normalize;
- filter abuse;
- aggregate counts;
- produce ranked candidates.

Historical popularity can be batch-computed.

Trending needs shorter windows.

## 4. Trie Mental Model

A trie maps prefixes to continuations.

```text
a
 └─p
    └─p
       ├─le
       └─lication
```

At each prefix node, store top K suggestions.

Then lookup:
```text
O(length of prefix)
```
conceptually plus return K.

Do not need implementation details.

## 5. Alternative — Sorted Strings / Prefix Range

Store sorted suggestion strings.

Prefix `"app"` corresponds to range:
```text
app...
```

Can use ordered index/search engine.

At moderate scale, DB/search index with prefix support may be simpler than custom trie.

## 6. Search Engine Option

A search engine can provide:
- prefix matching;
- ranking;
- typo/fuzzy;
- language analysis.

Use when requirements justify it.

Trie is a data structure concept, not mandatory architecture.

## 7. Precomputed Top K

The main latency trick:

Instead of:
```text
at query time scan all "app*" and rank
```

precompute:
```text
prefix "app" → top 10 suggestions
```

Store in:
- in-memory service;
- KV store;
- distributed cache;
- search index.

This shifts work from read to update pipeline.

## 8. Architecture

```text
User searches
   ↓
Event Stream
   ↓
Aggregation / Filtering
   ↓
Suggestion Builder
   ↓
Versioned Suggestion Index
   ↓
Autocomplete Service
   ↓
Cache / in-memory index
```

Read path should be tiny.

## 9. Ranking

Baseline score:
- query frequency;
- recency decay;
- region/language.

Example conceptual:
```text
score = historical_frequency × recency_weight
```

Exact ML formula unnecessary.

## 10. Trending

Historical index updates hourly may miss breaking trend.

Add realtime/near-realtime pipeline:

```text
recent query stream
   ↓
1/5-minute windows
   ↓
Trending Top-K
```

At read:
```text
base suggestions
+
trending candidates
→ merge/rerank
```

Need anti-abuse because bots can manipulate trends.

## 11. Personalization

User-level history:
```text
recent searches
clicked categories
locale
```

Do not rebuild global trie per user.

At read:
1. fetch global candidates;
2. fetch small personal candidate set;
3. rerank/boost;
4. filter privacy/safety.

Cache:
- global prefix heavily cacheable;
- personalized result less shareable.

Privacy:
- do not expose another user's queries;
- retention/deletion controls.

## 12. Prefix Explosion

For every query `"distributed systems"` prefixes:

```text
d
di
dis
...
```

Storing top K at every character can multiply storage.

Mitigations:
- minimum prefix length (e.g. 2/3);
- compressed trie;
- only common prefixes;
- search index;
- language-specific normalization.

Do not require custom trie if managed search meets workload.

## 13. Cache

Key:
```text
autocomplete:{locale}:{prefix}:{index_version}
```

Versioned index makes rollout easy.

Popular short prefixes are hot keys:
- local in-memory cache;
- replicate read service;
- edge caching for global nonpersonalized suggestions if safe.

## 14. Updating Index Safely

Build new version offline:

```text
v41 current
build v42
validate
atomic pointer switch → v42
```

Rollback to v41 if bad.

Avoid mutating massive in-memory trie inconsistently while serving.

Realtime trending layer can be separate overlay.

## 15. Failure

### Builder down
Serve last known index; freshness degrades.

### Trending pipeline down
Serve historical suggestions.

### Personalization store down
Serve global suggestions.

This is graceful degradation.

Autocomplete should not block core search.

## 16. Abuse/Safety

Bots can spam query terms to rank them.

Use:
- rate limits;
- trust weighting;
- minimum unique users;
- filtering/moderation;
- block unsafe suggestions.

Autocomplete suggestions themselves can become harmful content.

## Interview Questions

1. Why precompute top K?
2. Trie vs search engine?
3. How do trending suggestions stay fresh?
4. How do you personalize without per-user index?
5. What if builder fails?
6. How do you roll out a new index?
7. How do you prevent bot manipulation?
8. Why are very short prefixes hot?

## Common Mistakes

- Full DB scan per keystroke.
- Trie asserted without update/storage strategy.
- Personalization mixes users in shared cache.
- No trend-abuse defense.
- New index mutates serving structure in place.
- Autocomplete outage blocks search.
- No locale/normalization.
- No hot-prefix awareness.

## Must Remember

- **Autocomplete is a precomputed read problem.**
- **Prefix → top K should be cheap at request time.**
- **Trie is one option; sorted/search indexes may be simpler.**
- **Trending is a short-window overlay on historical ranking.**
- **Personalization reranks global candidates; do not build one index per user.**
- **Versioned index rollout simplifies consistency/rollback.**
- **Short prefixes are hot.**
- **Autocomplete should degrade without breaking search.**

## Self-Score

Use the 40-point rubric. If data pipeline or read-path explanation is weak, redraw both separately.
