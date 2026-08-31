# Session 1 — System Design Interview Framework

## 1. Must Learn

### Requirements & scope
- **Understand:** Distinguish functional requirements, non-functional requirements, constraints, scope, and non-goals; identify the few requirements that drive architecture.
- **Decision/trade-off:** Breadth vs focus: clarify enough to avoid designing the wrong system without spending the interview in requirements.

### Interview flow
- **Understand:** Know the sequence: requirements → scale → APIs/events → data model → high-level design → flows → deep dive → failures/scaling → trade-offs → summary.
- **Decision/trade-off:** Structure vs flexibility: follow a clear framework but adapt when the interviewer changes priorities.

### High-level architecture & flows
- **Understand:** Turn requirements into a small set of components and narrate one main read flow and one main write/async flow.
- **Decision/trade-off:** Simple coherent design vs premature complexity.

### Deep dives & bottlenecks
- **Understand:** Identify the hardest requirement or likely bottleneck and deepen only where it matters.
- **Decision/trade-off:** Depth on the critical path vs shallow coverage of everything.

### Trade-offs & failure handling
- **Understand:** For major choices, explain why, alternative, downside, and what happens when a dependency fails.
- **Decision/trade-off:** Performance/availability/correctness/cost choices should follow requirements, not buzzwords.

### Time management & collaboration
- **Understand:** Know how to allocate time, communicate assumptions, respond to hints, requirement changes, and close cleanly.
- **Decision/trade-off:** Leading the interview vs over-controlling it; enough detail vs finishing on time.

## 2. Should Know

- Back-of-the-envelope estimation as a design input; exact arithmetic is less important than architectural implication.
- API/event and data-model discussion at enough depth to make the architecture concrete.
- A short production-readiness pass: reliability, observability, security/privacy, cost when relevant.

## 3. Recognition Only

- HLD vs LLD expectations
- Product-oriented vs infrastructure-oriented prompts

## 4. Important Comparisons

- Functional vs non-functional requirements — behavior vs quality/constraints.
- Must-have vs nice-to-have requirements — design for the former first.
- Simple baseline vs prematurely distributed architecture — start simple unless scale/correctness forces complexity.
- Broad coverage vs deep dive — deepen the highest-risk requirement.

## 5. Important Interview Questions

1. What are the top 2–3 functional requirements and non-goals?
2. Which non-functional requirement dominates: latency, availability, durability, consistency, throughput, freshness, or cost?
3. What scale/skew matters enough to change the architecture?
4. What are the main read and write flows?
5. Which part of this design deserves the deep dive?
6. What breaks first at higher scale or during a dependency failure?
7. What are the biggest trade-offs and limitations of my design?

## 6. Common Interview Mistakes

- **Starting architecture immediately** → Clarify the requirements and design-driving constraints first.
- **Listing technologies without reasoning** → Tie every major component to a requirement or failure mode.
- **Overdesigning early** → Start with the simplest architecture that works, then evolve it.
- **Ignoring interviewer signals** → Treat questions/hints as evidence about where to deepen.
- **No closing summary** → Reserve time to restate decisions, trade-offs, and the biggest limitation.

## 7. Communication

### Important Vocabulary

functional requirement, non-functional requirement, scope, non-goal, assumption, bottleneck, invariant, trade-off, failure mode, read path, write path, deep dive

### Useful Interview Phrases

- “I’ll start with the simplest design that satisfies the current requirements.”
- “The requirement driving this choice is …”
- “The main trade-off here is …”
- “At 10× scale, I’d expect this component to become the first bottleneck.”

### Important Questions to Ask the Interviewer

- **Question:** “Which requirements are must-have for this interview?”  
  **Why it matters:** Prevents spending time on secondary features.
- **Question:** “Which quality matters most: latency, availability, consistency, or cost?”  
  **Why it matters:** Changes storage, replication, caching, and failure decisions.
- **Question:** “Should I optimize for current scale or discuss a 10× growth path?”  
  **Why it matters:** Controls how much scaling complexity is justified.

## 8. ⭐ Must Remember

1. Clarify before designing.
2. Let requirements drive components.
3. Start simple and evolve only when needed.
4. Narrate concrete read/write flows.
5. Deep-dive the hardest requirement, not every component.
6. Always discuss failure modes and trade-offs.
7. Save time for a concise closing summary.

## 9. Study Priority

1. Study first: requirements, scope/non-goals, and interview flow.
2. Study next: high-level architecture, read/write flows, and deep-dive selection.
3. Finish with: failure handling, trade-offs, collaboration, and closing summary.

## 10. Revision Checklist

- [ ] Run the full interview flow from memory.
- [ ] Turn requirements into a simple architecture and narrate key flows.
- [ ] Identify one bottleneck/failure and explain a reasonable evolution.
- [ ] Defend major choices with alternatives and trade-offs.
- [ ] Finish with a clear 2-minute summary.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
