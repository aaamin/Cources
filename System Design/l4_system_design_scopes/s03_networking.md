# Session 3 — Networking Fundamentals

## 1. Must Learn

### DNS & addressing
- **Understand:** Understand DNS resolving names to addresses and where lookup/caching failures add latency.
- **Decision/trade-off:** Caching/TTL and failover responsiveness vs lookup overhead/staleness.

### TCP vs UDP
- **Understand:** Know TCP as reliable ordered connection-oriented transport and UDP as lower-overhead datagrams without those guarantees.
- **Decision/trade-off:** Reliability/ordering vs latency/overhead depending on workload.

### TLS & HTTPS
- **Understand:** Know TLS provides encryption, server authentication, and integrity over network connections.
- **Decision/trade-off:** Security is mandatory for sensitive traffic; handshakes/termination affect latency and architecture.

### HTTP connections
- **Understand:** Understand request/response, keep-alive, connection reuse, and why repeated handshakes are expensive.
- **Decision/trade-off:** Connection reuse reduces latency/overhead but consumes connection resources.

### Latency, throughput & bandwidth
- **Understand:** Distinguish them and identify network hops/dependencies that add latency.
- **Decision/trade-off:** Fewer synchronous hops vs service decomposition; throughput vs per-request latency.

### Timeouts & network failure
- **Understand:** Understand partial failure: a request can be slow, dropped, or have an unknown outcome.
- **Decision/trade-off:** Waiting longer vs failing fast/retrying; retry can duplicate work.

## 2. Should Know

- Regional latency and why cross-region synchronous calls are expensive.
- Connection pools conceptually and why exhausting them can bottleneck services.
- Trace client → DNS → network/LB → app → database → response.

## 3. Recognition Only

- NAT
- HTTP/2 or HTTP/3 improvements at recognition depth
- MTU/packet loss only as low-level context

## 4. Important Comparisons

- TCP vs UDP — reliability/ordering vs lower transport overhead.
- HTTP vs HTTPS — plaintext vs TLS-protected HTTP.
- Latency vs throughput — time per operation vs work per unit time.
- New connection vs connection reuse — handshake cost vs persistent resource use.
- In-region vs cross-region calls — lower latency/failure exposure vs geographic separation.

## 5. Important Interview Questions

1. What happens between entering a hostname and the request reaching the service?
2. Why does connection reuse improve performance?
3. Where can network latency accumulate in a multi-service request?
4. What should a client do when a request times out but may have succeeded?
5. Why are long synchronous dependency chains fragile?

## 6. Common Interview Mistakes

- **Treating network calls like local function calls** → Account for latency, timeout, partial failure, and retries.
- **Confusing bandwidth and latency** → Discuss them as different limits.
- **Ignoring TLS** → Assume production traffic needs encryption/authentication unless there is a strong reason otherwise.
- **No timeout strategy** → Bound waits and define failure behavior.
- **Cross-region calls casually** → Call out latency and larger failure surface.

## 7. Communication

### Important Vocabulary

DNS, IP address, TCP, UDP, TLS, HTTP, HTTPS, connection, keep-alive, connection reuse, latency, throughput, bandwidth, timeout, network hop

### Useful Interview Phrases

- “This is a network boundary, so I need to account for timeout and partial failure.”
- “I’d reuse connections to avoid repeated handshake overhead.”
- “Cross-region synchronous calls add both latency and failure exposure.”

### Important Questions to Ask the Interviewer

- **Question:** “Is traffic global or mostly region-local?”  
  **Why it matters:** Affects routing and cross-region latency.
- **Question:** “Are requests latency-sensitive or throughput-oriented?”  
  **Why it matters:** Changes protocol and batching choices.
- **Question:** “Can a timed-out operation be safely retried?”  
  **Why it matters:** Determines idempotency requirements.

## 8. ⭐ Must Remember

1. Network calls can be slow, lost, duplicated through retries, or have unknown outcomes.
2. TCP provides reliable ordered delivery; UDP does not.
3. HTTPS is HTTP over TLS.
4. Reuse connections when appropriate.
5. Latency and throughput are different.
6. Every synchronous network hop adds latency and failure risk.

## 9. Study Priority

1. Study first: DNS, TCP/UDP, TLS, HTTP.
2. Study next: connections, reuse, latency/throughput/bandwidth.
3. Finish with: timeouts, request tracing, and regional latency.

## 10. Revision Checklist

- [ ] Trace a request end-to-end.
- [ ] Explain TCP vs UDP and HTTP vs HTTPS.
- [ ] Explain connection reuse and timeouts.
- [ ] Identify latency/failure points in a request chain.
- [ ] Discuss cross-region call trade-offs.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
