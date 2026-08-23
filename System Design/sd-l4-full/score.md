# 40-Point Mock Score Sheet

Score each category from 0 to 4.

| Category | Score | What strong performance means |
|---|---:|---|
| Requirements & scope | /4 | Priorities, constraints, non-goals are controlled |
| Estimation & workload | /4 | Peak/skew/scale are tied to decisions |
| APIs / events / data | /4 | Access-pattern driven; errors/idempotency/ownership clear |
| HLD & flows | /4 | Simple complete architecture; read/write/async flows narrated |
| Scalability | /4 | Workload-specific bottlenecks, hot spots, 10× evolution |
| Correctness | /4 | Invariants, ordering, races, duplicates, recovery |
| Reliability / operations | /4 | Failure domains, degradation, observability, deploy/recovery |
| Security / privacy / cost | /4 | Relevant risks and major cost drivers prioritized |
| Trade-offs / evolution | /4 | Credible alternative and justified evolution path |
| Communication / time | /4 | Leads clearly, adapts, finishes and summarizes |
| **Total** | **/40** | |

## Interpretation

- **34–40:** strong L4 signal for this prompt
- **32–33:** passing readiness signal
- **28–31:** close, but meaningful risk remains
- **24–27:** foundations exist; integration needs repair
- **below 24:** revisit relevant fundamentals

A 0 or 1 in requirements, API/data, HLD, correctness, trade-offs, or communication is a non-pass regardless of total.

## Repair log

```text
Weak category #1:
Weak category #2:

Three biggest misses:
1.
2.
3.

Narrow drill #1:
Narrow drill #2:

What I will test on the next unseen mock:
```
