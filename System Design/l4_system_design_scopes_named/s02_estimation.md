# Session 2 — Back-of-the-Envelope Estimation

## 1. Must Learn

### Traffic estimation
- **Understand:** Estimate DAU/MAU only when useful, then derive average and peak QPS plus read/write ratio.
- **Decision/trade-off:** Average vs peak provisioning; optimize for meaningful peak/skew rather than averages alone.

### Data volume & storage growth
- **Understand:** Estimate payload/object size, writes per day, retention, and rough storage growth.
- **Decision/trade-off:** Retention/durability vs storage cost; database vs object-storage implications.

### Bandwidth
- **Understand:** Estimate ingress/egress when payloads or media are large.
- **Decision/trade-off:** Network/CDN cost and bottlenecks vs backend compute/storage.

### Concurrency
- **Understand:** Estimate simultaneous users or persistent connections when the design needs connection capacity.
- **Decision/trade-off:** Request QPS vs connection-count scaling require different infrastructure.

### Skew, bursts & headroom
- **Understand:** Account for hot users/keys, burst traffic, growth, and safety margin.
- **Decision/trade-off:** Efficiency vs capacity headroom; average utilization vs resilience to spikes.

## 2. Should Know

- Use powers-of-ten arithmetic and state assumptions; interview accuracy matters more than precision.
- Translate every important estimate into a component or capacity implication.
- Recognize when an estimate is unnecessary and skip it.

## 3. Recognition Only

- MAU/DAU ratio as engagement signal
- Storage replication multiplier
- Compression as a rough cost modifier

## 4. Important Comparisons

- Average QPS vs peak QPS — peak usually drives capacity.
- QPS vs concurrent connections — throughput vs simultaneously open state.
- Logical data size vs physical storage — replicas/indexes/overhead increase physical footprint.
- Precision vs speed — rough correct order-of-magnitude beats detailed irrelevant math.

## 5. Important Interview Questions

1. Which estimate would actually change my design?
2. How do I derive peak QPS from usage assumptions?
3. When does storage volume push data toward object storage or partitioning?
4. When do concurrent connections become more important than request QPS?
5. How should traffic skew and bursts change capacity planning?

## 6. Common Interview Mistakes

- **Calculating everything** → Estimate only numbers tied to decisions.
- **Using average traffic only** → State peak/burst assumptions and headroom.
- **Giving numbers with no consequence** → Immediately state what each estimate implies architecturally.
- **False precision** → Use rounded assumptions and order-of-magnitude math.
- **Ignoring skew** → Call out hot keys/users/tenants when they could dominate load.

## 7. Communication

### Important Vocabulary

DAU, MAU, average QPS, peak QPS, read/write ratio, payload size, bandwidth, concurrency, retention, skew, burst, headroom

### Useful Interview Phrases

- “I only need an order-of-magnitude estimate here.”
- “The important number is peak QPS because it drives capacity.”
- “This estimate matters because it changes …”
- “I’ll include headroom for bursts rather than size exactly to the average.”

### Important Questions to Ask the Interviewer

- **Question:** “What peak-to-average ratio should I assume?”  
  **Why it matters:** Changes provisioning and overload strategy.
- **Question:** “How long must the data be retained?”  
  **Why it matters:** Directly changes storage size/cost.
- **Question:** “Are users continuously connected or request/response only?”  
  **Why it matters:** Determines whether connection capacity matters.

## 8. ⭐ Must Remember

1. Estimate only what can change a design decision.
2. Peak load matters more than average load for capacity.
3. State assumptions explicitly.
4. Always connect a number to an architectural implication.
5. Include skew/bursts/headroom when relevant.
6. Order-of-magnitude correctness is enough.

## 9. Study Priority

1. Study first: QPS, read/write ratio, peak traffic.
2. Study next: storage, bandwidth, and concurrency.
3. Finish with: skew, bursts, headroom, and implication-driven estimation.

## 10. Revision Checklist

- [ ] Estimate peak QPS from usage assumptions.
- [ ] Estimate storage growth and bandwidth when relevant.
- [ ] Distinguish QPS from concurrent connections.
- [ ] Explain the architecture implication of each important estimate.
- [ ] Avoid irrelevant or over-precise calculations.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
