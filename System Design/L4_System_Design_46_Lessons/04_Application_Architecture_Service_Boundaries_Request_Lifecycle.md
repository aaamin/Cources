# Session 4 — Application Architecture, Service Boundaries & Request Lifecycle

## Outcome

You should be able to choose between a monolith, modular monolith, and microservices; identify sensible service boundaries; explain database ownership and sync/async communication; and recognize the operational costs introduced by distribution.

## Why This Matters

“Use microservices” is not a scaling strategy by itself.

A well-structured monolith can serve very large workloads when horizontally replicated. A badly split microservice architecture can be slower, less reliable, harder to deploy, and much harder to reason about.

At L4, the interviewer wants to see that you introduce service boundaries because a requirement justifies them—not because microservices sound modern.

## Mental Model

Architecture choice is a trade-off among:

```text
Domain boundaries
Team ownership
Independent deployment
Independent scaling
Data consistency
Failure isolation
Operational complexity
System maturity
```

## Monolith

A monolith packages the application as one deployable unit.

Conceptually:

```text
Client
  ↓
Load Balancer
  ↓
Application
 ├─ Users
 ├─ Orders
 ├─ Payments
 └─ Notifications
  ↓
Database
```

### Advantages

- simple local calls;
- easier transactions;
- one deployment/runtime topology;
- straightforward debugging;
- fewer distributed failure modes;
- easier development for smaller teams.

### Disadvantages

As the system/team grows:
- modules may become tightly coupled;
- deploys affect the whole application;
- one hot feature may require scaling the entire process;
- build/test/deploy cycles may slow;
- ownership may become unclear.

A monolith is not automatically a “single server.” You can run many stateless copies behind a load balancer.

## Modular Monolith

A modular monolith stays one deployable application but enforces strong internal boundaries.

```text
Application
 ├─ User Module
 ├─ Catalog Module
 ├─ Order Module
 └─ Payment Module
```

Each module should:
- own its business logic;
- expose explicit interfaces;
- avoid reaching into another module's internals;
- ideally make later extraction possible.

This is often a strong default for a new product: boundaries without paying all distributed-system costs.

## Microservices

Microservices split capabilities into independently deployable services.

```text
Client
  ↓
Gateway
  ├─ User Service → User DB
  ├─ Order Service → Order DB
  └─ Payment Service → Payment DB
```

### Potential benefits

- independent deployment;
- independent scaling;
- clearer ownership;
- fault isolation when designed well;
- freedom to evolve services separately.

### Costs

- every cross-service call can fail;
- distributed tracing/observability becomes necessary;
- cross-service transactions are difficult;
- versioning contracts matters;
- deployment orchestration increases;
- testing becomes more complex;
- eventual consistency appears;
- operational overhead increases.

Microservices are a trade: local complexity can become distributed complexity.

## When to Split a Service

Strong reasons include:

### Independent scale
Video transcoding needs massive compute while user-profile reads do not.

### Distinct availability/failure domain
An analytics subsystem should not bring down checkout.

### Separate business boundary
Payments have their own invariants, security requirements, and ownership.

### Independent deployment/ownership
A large organization has separate teams with distinct release cadences.

### Technology/resource shape
Search indexing, realtime connection gateways, or media processing may have very different runtime needs.

Weak reasons:
- “the table is large”;
- “microservices scale better”;
- “FAANG uses microservices”;
- one service per database table.

## Service Boundaries

A service boundary should usually align with cohesive business responsibility.

Bad decomposition:

```text
UserNameService
UserEmailService
UserAddressService
```

This creates chatty calls and weak cohesion.

Better:

```text
Identity/Profile
Orders
Payments
Notifications
Catalog
```

Exact boundaries depend on the domain.

Ask:
- Which data/invariant belongs together?
- Which operations need one transaction?
- Which parts scale independently?
- Who owns the capability?
- How often do modules call each other?

## Database Ownership

A microservice should ideally own its data.

Why?

If Order Service and Payment Service directly update each other's tables:
- schema changes couple teams;
- invariants are unclear;
- services are not really independent;
- deploys become coordinated.

### Database-per-service concept

This does **not** necessarily mean a dedicated physical database server per service. It means logical ownership: other services interact through a contract, not arbitrary SQL against private tables.

## Shared Database Trade-Off

A shared DB can be perfectly reasonable in a monolith/modular monolith because:
- joins are convenient;
- transactions are simple;
- operations are easier.

But with independently deployed services, a shared schema can create tight coupling.

Do not force database-per-service before you actually need service independence.

## Synchronous Service Communication

Examples:
- HTTP/REST;
- gRPC.

Good when:
- caller needs result immediately;
- operation is request/response;
- dependency latency is acceptable.

Costs:
- caller is coupled to dependency availability/latency;
- long chains increase tail latency;
- retries can amplify load.

Example:

```text
Checkout → Order → Inventory → Payment → Fraud → Email
```

Keeping email in this synchronous chain is unnecessary and risky.

## Asynchronous Communication

Examples:
- queue;
- Pub/Sub;
- stream/event log.

Good when:
- caller does not need immediate result;
- work can be retried;
- fan-out to many consumers;
- buffering bursts;
- workflows tolerate eventual completion.

Example:

```text
OrderCreated
   ↓
Event Bus
 ├─ Email
 ├─ Analytics
 └─ Loyalty Points
```

Costs:
- eventual consistency;
- duplicate handling;
- observability/debugging complexity;
- ordering concerns.

## Request Lifecycle

For a synchronous endpoint:

```text
Client
 ↓
Gateway/LB
 ↓
Application
 ↓
Acquire DB connection
 ↓
Query DB
 ↓
Maybe call downstream
 ↓
Serialize response
 ↓
Return
```

At each stage latency can accumulate.

A request may wait for:
- load-balancer queue;
- application thread/task resources;
- connection pool;
- DB lock;
- downstream API;
- network.

## Stateful vs Stateless Services

### Stateless application service
Any replica can handle any request because durable/session state lives elsewhere.

Advantages:
- easy horizontal scaling;
- easy replacement;
- simple load balancing.

### Stateful service
A particular node owns in-memory/session/connection state.

Examples:
- WebSocket connection gateway;
- stateful stream processor.

Stateful services are not wrong, but scaling/rebalancing/failover require more design.

## Independent Scaling

Suppose:
- image processing needs 100× CPU;
- profile service has light reads.

With a monolith, scaling the whole application for image processing may be wasteful.

Possible solution:
- keep core business system modular;
- extract processing workers/service because their resource pattern differs.

This is requirement-driven decomposition.

## Distributed Failure Cost

Local function:

```text
result = pricing.Calculate(order)
```

Remote function:

```text
result = pricingService.Calculate(order)
```

Remote call adds:
- serialization;
- network;
- timeout;
- authentication;
- retries;
- version compatibility;
- partial failure;
- observability needs.

Treat every service split as creating a new failure boundary.

## Worked Example — E-commerce Startup

Initial requirements:
- catalog;
- cart;
- checkout;
- orders;
- payment provider;
- email notifications.

A reasonable starting architecture:

```text
LB
 ↓
Modular Monolith
 ├─ Catalog
 ├─ Cart
 ├─ Orders
 └─ Checkout
 ↓
Relational DB

OrderCompleted → Queue → Notification Worker
Payment provider is external
```

Why not five microservices immediately?
- one team;
- moderate traffic;
- strong transactional relationships;
- independent scaling is not yet necessary.

Later:
- catalog traffic becomes huge → cache/search read path may separate;
- image processing → separate workers;
- payments → dedicated service due to security/ownership;
- notification → already async and independently scalable.

The architecture evolves because needs appear.

## Comparison Table

| Property | Monolith | Modular Monolith | Microservices |
|---|---|---|---|
| Deployable units | One | One | Many |
| Local transactions | Easy | Easy | Cross-service difficult |
| Network failures internally | Minimal | Minimal | Common concern |
| Independent scaling | Limited | Limited | Strong |
| Independent deployment | No | No | Yes |
| Operational complexity | Lowest | Low | Highest |
| Boundary discipline | Often weak | Explicit | Explicit by network/API |
| Good default for small/new system | Often | **Often strongest** | Only with justification |

## Small Design Drills

1. A five-person team is building a new CRUD-heavy SaaS product. Why might a modular monolith be better than microservices?
2. Video transcoding consumes 100× CPU compared with metadata APIs. What might you extract?
3. Order and payment data require one strongly consistent local transaction. What becomes harder if split too early?
4. Why is “one microservice per table” usually poor decomposition?
5. Is a monolith limited to one machine?
6. An email provider is slow. Should checkout wait for email sending?

<details>
<summary>Answer key</summary>

1. Simpler operations/transactions/deployments while preserving domain boundaries.
2. Transcoding workers/service because it has different scaling/resource requirements.
3. Atomic cross-service transaction; you may need Saga/outbox/reconciliation instead of one DB transaction.
4. Tables are storage details, not necessarily cohesive business capabilities; it creates chatty coupling.
5. No. Stateless monolith replicas can horizontally scale behind a load balancer.
6. Usually no. Queue notification work after the durable business action.

</details>

## Interview Questions

1. Monolith vs microservices: what are the trade-offs?
2. What is a modular monolith?
3. When would you extract a service?
4. What does database ownership mean?
5. Why are long synchronous service chains dangerous?
6. When should communication be asynchronous?
7. Are stateless services always better?
8. Why don't microservices automatically make a system scalable?

## Common Mistakes

- Treating monolith as one physical server.
- Automatically choosing microservices for “large scale.”
- Splitting by database table.
- Ignoring data ownership.
- Ignoring distributed transaction complexity.
- Making non-critical work synchronous.
- Drawing 15 services without explaining why.
- Assuming service boundaries eliminate coupling.
- Sharing every database table across supposedly independent services.

## Must Remember

- **Microservices are an organizational/architectural trade-off, not a default scaling trick.**
- **A monolith can horizontally scale.**
- **A modular monolith is often an excellent starting point.**
- **Split around cohesive business/resource boundaries.**
- **Every remote call introduces latency and partial failure.**
- **Database ownership matters for true service independence.**
- **Use async communication when immediate completion is unnecessary.**
- **Do not introduce distributed transactions without need.**
- **Evolve architecture when requirements justify it.**

## Interview Revision Summary

Decision questions:

```text
Do we need independent deployment?
Do we need independent scaling?
Is there a clear domain boundary?
Does one team own everything?
Do operations need one transaction?
Will splitting create chatty calls?
Can we tolerate eventual consistency?
Can we operate/debug many services?
```

If the answers do not justify distribution, keep it simpler.

## Explain Without Notes

Explain why you might start an e-commerce product as a modular monolith and later extract notifications, search, media processing, and payments.

## Completion Checklist

- [ ] I can compare monolith/modular monolith/microservices.
- [ ] I can explain good service-boundary signals.
- [ ] I understand database ownership.
- [ ] I can choose sync vs async communication.
- [ ] I can explain distributed failure cost.
- [ ] I do not equate microservices with scalability automatically.
