# Microservices

## Monolithic Architecture

### How it works

A monolithic application packages all functionality – business logic, database access, messaging, UI – into a single deployable unit (e.g., a WAR file, a Rails directory hierarchy).

![monolithic-architecture.png](../images/microservices/monolithic-architecture.png)

At the core is the business logic, surrounded by adapters that interface with the outside world: database access components, messaging components, and web/API components. Despite being logically modular, the entire application is deployed as one unit.

### Benefits of Monolithic Architecture

- **Simple to develop** – IDEs and tooling are built around single-application development.
- **Simple to test** – end-to-end tests can launch the whole app and test the UI directly.
- **Simple to deploy** – copy the packaged artifact to a server.
- **Simple to scale** – run multiple identical copies behind a load balancer.

Works well in the early stages of a project.

### Drawbacks of Monolithic Architecture (Monolithic Hell)

As the application grows, the simplicity disappears:

- **Overwhelming complexity** – no single developer can fully understand the codebase. Bugs are hard to fix and features hard to add correctly. A downward spiral: hard to understand → changes made incorrectly → quality declines.
- **Slow IDE and startup times** – large codebases slow down tooling. Startup times of 12–40 minutes have been reported, significantly hurting developer productivity.
- **Continuous deployment is blocked** – any change requires redeploying the entire application, increasing risk and discouraging frequent releases.
- **Limited scalability** – can only scale in one dimension (more copies). Different modules with conflicting resource needs (CPU-intensive vs. memory-intensive) must run on the same hardware.
- **Reliability risk** – a bug (e.g., a memory leak) in one module can bring down the entire process for all users.
- **Technology lock-in** – rewriting even one part to use a newer framework often requires rewriting the entire application.

---

## Microservice Architecture

### How it works

Instead of one large application, the system is split into a set of small, independently deployable services. Each service:

- Implements a focused area of functionality (e.g., order management, customer management).
- Has its own hexagonal architecture (business logic and adapters).
- Has its own database (database-per-service pattern).
- Communicates via REST APIs or asynchronous messaging.

![microservices-architecture.png](../images/microservices/microservices-architecture.png)

Each service instance typically runs as a Docker container on a cloud VM. An **API Gateway** sits in front, handling load balancing, caching, access control, metering, and monitoring – clients never talk directly to backend services.

### The Scale Cube

Microservices correspond to **Y-axis scaling** on the Scale Cube:

![scale-cube.png](../images/microservices/scale-cube.png)

| Axis       | Description                                                                           |
|------------|---------------------------------------------------------------------------------------|
| **X-axis** | Run multiple identical copies behind a load balancer.                                 |
| **Y-axis** | Decompose the application by function – microservices.                                |
| **Z-axis** | Partition data by request attribute (e.g. customer ID) to route to a specific server. |

Applications typically use all three together.

### Deployment example (Docker on EC2)

![dockerized-application.png](../images/microservices/dockerized-application.png)

Multiple instances of a service run as Docker containers across cloud VMs, with a load balancer (e.g., NGINX) in front distributing traffic.

### Database architecture

Each service owns its own database schema – **polyglot persistence**: each service can use the database type best suited to its needs (e.g., a geo-query-optimized database for driver location lookups).

![intro-microservices.png](../images/microservices/intro-microservices.png)

This approach causes some data duplication and conflicts with a unified enterprise data model, but is essential for loose coupling between services.

### Benefits of Microservice Architecture

- **Tackles complexity** – each service is small, has a well-defined API boundary, and is easier to understand, develop, and maintain.
- **Independent development** – each service can be owned and developed by a small, focused team using whatever technology stack makes sense.
- **Independent deployment** – services are deployed independently; no coordination required for changes local to one service. Enables true continuous deployment.
- **Independent scaling** – each service can be scaled to meet its own capacity needs on hardware that matches its resource requirements (e.g., CPU-optimized for image processing, memory-optimized for caching).
- **Improved fault isolation** – a failure in one service does not cascade to bring down the entire system.
- **No long-term technology commitment** – new services can be written with new technology; old services can be rewritten when needed.

### Drawbacks of Microservice Architecture

- **Distributed system complexity** – inter-process communication, partial failure handling, and distributed transactions all require explicit design and code.
- **Partitioned database** – cross-service business transactions cannot use a single ACID transaction. Requires eventual consistency approaches (e.g., the Saga pattern).
- **Testing complexity** – testing a service requires launching it along with all its dependencies (or stubs for them).
- **Cross-service changes require coordination** – a change spanning services A → B → C must be carefully planned and rolled out in order.
- **Deployment complexity** – many services, each with multiple instances, need to be configured, deployed, scaled, and monitored. Requires mature automation (Kubernetes, Docker, PaaS).
- **Increased memory consumption** – N monolithic instances become N×M service instances, each with its own runtime overhead.

---

## Monolithic vs. Microservices – Summary

| Concern                | Monolithic                          | Microservices                              |
|------------------------|-------------------------------------|--------------------------------------------|
| Development simplicity | High (initially)                    | Lower – distributed system overhead        |
| Deployment             | Single artifact                     | Many independent deployments               |
| Scalability            | X-axis only (copies)                | X + Y + Z axes                             |
| Fault isolation        | Poor – one bug can crash everything | Good – failures are contained              |
| Technology flexibility | Low – stack locked in at start      | High – per-service choice                  |
| Continuous deployment  | Hard – full redeploy required       | Easy – deploy one service at a time        |
| Testing                | Simple end-to-end                   | Complex – must manage service dependencies |
| Suitable for           | Simple, early-stage applications    | Complex, evolving, large-team applications |

---

## Service-Oriented Architecture (SOA)

SOA is a design paradigm where software components behave as **separate, autonomous, loosely coupled, network-accessible units** that communicate via standardized protocols.

![Service-Oriented-Architecture.png](../images/microservices/Service-Oriented-Architecture.png)

**Loose coupling** means a client service can communicate with another service without being tightly bound to it – it only depends on a defined interface, not the implementation, language, or platform behind it.

**Key drivers of SOA:**

- **Distributed systems** – components are spread across networks; standardized interfaces (APIs, OSI protocols) allow them to communicate regardless of vendor or technology.
- **Ownership limitations** – in cloud environments, customers cannot modify cloud infrastructure; services must interoperate without requiring control of underlying components.
- **Heterogeneity** – large systems are built over time with different languages, frameworks, and platforms. SOA enables interoperability across this diversity, avoiding vendor lock-in.

**Microservices vs. SOA:** Microservices can be seen as SOA without the heavyweight WS-* specifications and Enterprise Service Bus (ESB). Microservices favor lightweight REST over WS-*, implement ESB-like functionality within individual services, and reject the concept of a canonical schema.

---

## Inter-Service Communication

Services need to talk to each other, and the communication style is an architectural decision with real consequences for coupling, availability, and complexity.

**Synchronous communication**

The caller sends a request and blocks until it receives a response. Common protocols are REST over HTTP and gRPC (which uses HTTP/2 and Protocol Buffers for efficiency). Synchronous calls are simple to reason about but introduce temporal coupling — if the downstream service is slow or unavailable, the caller is directly affected. This is the failure mode that the Circuit Breaker pattern addresses.

**Asynchronous messaging**

The caller publishes a message to a broker (e.g. Kafka, RabbitMQ) and moves on without waiting for a response. The receiving service consumes the message in its own time. This decouples services in both time and availability — the producer does not need the consumer to be running, and backlogs can be absorbed by the queue. The tradeoff is added infrastructure complexity and eventual consistency: the system will reach a consistent state, but not immediately.

**Choosing between them**

Use synchronous communication when you need an immediate response and both services are expected to be available (e.g. a query that must return data to a user). Use asynchronous messaging when you can tolerate latency, need to broadcast an event to multiple consumers, or want to protect a service from being overwhelmed by upstream load.

---

## API Gateway

The API Gateway is the single entry point for all client traffic. Rather than clients knowing about and calling individual services directly, all requests go through the gateway, which routes them to the appropriate service.

**Responsibilities:**

- **Request routing** – maps incoming requests to the correct downstream service.
- **Authentication and authorization** – verifies tokens (e.g. JWT, OAuth2) before a request reaches any service, so individual services do not need to implement auth logic themselves.
- **Rate limiting and throttling** – enforces a maximum number of requests a client can make within a time window (e.g. 1000 requests per minute per API key). Requests that exceed the limit are rejected with a `429 Too Many Requests` response before they reach any downstream service. Common algorithms include the token bucket (allows controlled bursting) and the fixed window counter (simpler but can allow double the limit at window boundaries).
- **Backpressure** – where rate limiting protects services from individual misbehaving clients, backpressure protects the system from being overwhelmed in aggregate. When downstream services are running close to capacity, the gateway can actively slow down or shed incoming traffic rather than allowing requests to pile up and cause cascading failures. This is typically implemented by monitoring downstream queue depths or response latencies and rejecting excess requests with a `503 Service Unavailable` once thresholds are crossed. The principle is to fail fast at the edge rather than let pressure propagate inward.
- **SSL termination** – handles TLS at the gateway so internal service-to-service communication can use plain HTTP.
- **Request/response transformation** – can reshape payloads, aggregate responses from multiple services into one, or translate between protocols (e.g. REST to gRPC).
- **Observability** – a natural point to collect metrics, logs, and trace IDs for every request entering the system.

**Tradeoff:** The gateway becomes a critical component — if it goes down, no traffic reaches any service. It must be highly available and kept operationally simple to avoid becoming a bottleneck.

---

## Service Discovery

In a microservices system, service instances are created and destroyed dynamically (scaled up, redeployed, crashed and restarted). Hardcoding IP addresses is not viable; services need a way to find each other at runtime.

**Client-side discovery**

The calling service queries a **service registry** (e.g. Eureka, Consul) to get the list of available instances for a target service, then picks one using a load balancing algorithm (e.g. round-robin). The client is responsible for both the lookup and the balancing decision.

**Server-side discovery**

The calling service sends its request to a load balancer or router (often the API Gateway), which queries the registry and forwards the request to an available instance. The calling service has no awareness of the registry or the instance selection logic.

**Service registry**

Both approaches depend on a registry that services register with on startup and deregister from on shutdown (or are removed from when health checks fail). Consul and Eureka maintain this registry and expose it via an API. In Kubernetes, this role is filled natively by the cluster's internal DNS and the control plane.

---

## Load Balancing

Load balancing distributes incoming traffic across multiple service instances to maximize throughput, minimize latency, and avoid overloading any single instance. It operates at different layers of a microservices system — at the edge (API Gateway), between services, and within the orchestration platform.

### Load Balancing Algorithms

**Round Robin**
Requests are distributed to instances in a fixed cyclic order. Simple and predictable, but takes no account of instance state — a slow or heavily loaded instance receives the same share of traffic as a healthy one. Works well when instances are homogeneous and request cost is roughly uniform.

**Weighted Round Robin**
Each instance is assigned a weight, and traffic is distributed proportionally. Useful when instances have different capacities (e.g. during a canary deployment where the new version receives 10% of traffic and the old version receives 90%).

**Least Connections**
Each new request is routed to the instance with the fewest active connections at that moment. Better than round robin when request processing time varies significantly, since it naturally directs traffic away from instances that are already busy handling long-running requests.

**Least Response Time**
Routes to the instance with the lowest combination of active connections and observed response latency. More adaptive than least connections alone, but requires the load balancer to track response time measurements per instance.

**IP Hash / Consistent Hashing**
A hash of the client's IP address (or another request attribute, such as a session ID or user ID) determines which instance handles the request. Requests from the same client consistently hit the same instance. This is useful when instance-local state matters — for example, in-memory caches or session data — but creates uneven distribution if client traffic is skewed, and requires rehashing when instances are added or removed. Consistent hashing mitigates the rehashing problem by minimizing the number of requests that change instance when the pool size changes.

**Random**
An instance is chosen at random. Counterintuitively, this performs close to round robin in aggregate due to the law of large numbers, and requires no state in the load balancer. Sometimes used as a baseline or for low-traffic internal routes.

### Layer 4 vs. Layer 7 Load Balancing

Load balancers operate at different levels of the network stack, and the distinction has practical consequences.

**Layer 4 (Transport Layer)** load balancers route based on TCP/UDP connection metadata — source IP, destination IP, and port. They do not inspect the payload. This makes them extremely fast and suitable for raw throughput, but they cannot make routing decisions based on HTTP headers, URL paths, or cookies.

**Layer 7 (Application Layer)** load balancers inspect the full HTTP request. This enables content-based routing — directing `/api/orders` to one service and `/api/customers` to another — as well as header-based routing, SSL termination, and cookie-based session affinity. The API Gateway in a microservices system is a Layer 7 load balancer. The cost is slightly higher per-request overhead compared to Layer 4.

### Health Checking and Instance Removal

A load balancer is only useful if it routes traffic to healthy instances. Load balancers continuously probe instances using health checks — either passive (watching for error responses on live traffic) or active (sending periodic probe requests to a health endpoint). An instance that fails a configured number of consecutive checks is removed from the pool and stops receiving traffic until it recovers.

In Kubernetes, the platform's liveness and readiness probes serve this role: a pod that fails its readiness probe is removed from the Service endpoint list and receives no traffic, even if the process is still running.

### Relationship to Other Patterns

Load balancing does not operate in isolation. It works alongside service discovery — the load balancer needs an up-to-date list of available instances to route to, which it obtains from the service registry or the orchestration platform. It also intersects with the Circuit Breaker pattern: while the load balancer routes away from instances that fail health checks, the Circuit Breaker stops calls to a downstream service entirely when the failure rate crosses a threshold, regardless of which instance is selected.

---

## Containerization and Orchestration

### Containers

A container packages an application together with everything it needs to run — the runtime, libraries, and configuration — into a single isolated unit. Unlike a virtual machine, a container shares the host OS kernel rather than virtualizing an entire operating system, making it lightweight and fast to start.

For microservices, containers matter because they eliminate environment inconsistency. The same image that a developer builds locally is what gets deployed to staging and production. Each service runs in its own container, so dependency conflicts between services are impossible.

Docker is the standard container runtime. A service is packaged as a **Docker image** (built from a `Dockerfile`) and run as a **container** — an instance of that image.

### Orchestration

Running a handful of containers manually is manageable. Running hundreds of service instances across many machines — with auto-scaling, self-healing, rolling deployments, and service discovery — is not. That is what an orchestrator handles.

**Kubernetes** is the dominant orchestration platform. Key concepts:

- **Pod** – the smallest deployable unit; typically wraps a single container.
- **Deployment** – declares how many replicas of a pod should be running and manages rolling updates.
- **Service** – a stable DNS name and virtual IP that load balances traffic across healthy pods; solves service discovery natively.
- **ConfigMap / Secret** – separates configuration and credentials from container images.
- **Horizontal Pod Autoscaler** – automatically scales the number of pod replicas based on CPU, memory, or custom metrics.

Kubernetes continuously reconciles desired state with actual state — if a pod crashes, it is restarted automatically; if a node fails, pods are rescheduled elsewhere.

---

## Security

Security in a microservices system has two distinct concerns: authenticating external clients at the edge, and securing communication between services internally.

**Edge authentication**

The API Gateway is the right place to authenticate incoming requests. Clients obtain a token (typically a JWT signed by an identity provider using OAuth2 / OpenID Connect) and include it in requests. The gateway validates the token's signature and expiry before forwarding the request. Individual services receive the validated identity in a header and do not need to reimplement auth logic.

**Service-to-service authentication**

Internal services should not blindly trust any request that arrives on the internal network. Two common approaches:

- **JWT propagation** – the original user's JWT is forwarded through the call chain. Each service can validate it and extract the caller's identity. Simple but ties internal calls to user sessions.
- **Mutual TLS (mTLS)** – each service has its own certificate. When service A calls service B, both sides present and verify certificates, ensuring neither can be impersonated. This is network-level identity independent of the user session. Service meshes (e.g. Istio, Linkerd) can handle mTLS transparently without requiring services to implement it themselves.

**Secrets management**

Credentials (database passwords, API keys, signing keys) should never be baked into container images or stored in source control. They should be injected at runtime via a secrets manager (e.g. HashiCorp Vault, AWS Secrets Manager) or Kubernetes Secrets.

---

## Patterns

### Saga

#### Context

In a microservices architecture with database-per-service, some business transactions span multiple services. These cannot use a single local ACID transaction.

#### Problem

How to implement transactions that span multiple services? (2PC — a two-phase commit — is not an option.)

#### Solution

Implement each multiservice business transaction as a **saga** – a sequence of local transactions. Each local transaction updates its own database and publishes an event or message to trigger the next step. If a step fails, the saga executes **compensating transactions** to undo the changes made by preceding steps.

![From_2PC_To_Saga.png](../images/microservices/From_2PC_To_Saga.png)

#### Two coordination approaches

**1. Choreography** – each service listens for events and reacts by executing its local transaction and emitting the next event. No central coordinator.

![Create_Order_Saga.png](../images/microservices/Create_Order_Saga.png)

Example flow (order creation):
1. Order Service receives `POST /orders` → creates Order in `PENDING` state → emits `OrderCreated` event.
2. Customer Service reserves credit → emits outcome event.
3. Order Service approves or rejects the Order based on the outcome.

**2. Orchestration** – a central saga orchestrator tells each service what to do and handles the overall flow.

![Create_Order_Saga_Orchestration.png](../images/microservices/Create_Order_Saga_Orchestration.png)

Example flow (order creation):
1. Order Service receives `POST /orders` → creates the saga orchestrator.
2. Orchestrator creates Order in `PENDING` state → sends `ReserveCredit` command to Customer Service.
3. Customer Service attempts to reserve credit → replies with an outcome.
4. Orchestrator approves or rejects the Order.

---

### Circuit Breaker

#### Context

In synchronous inter-service communication, a slow or unavailable downstream service can cause the calling service to exhaust its thread pool waiting for responses, eventually cascading the failure to other services.

#### Problem

How to prevent a network or service failure from cascading to other services?

#### Solution

Wrap remote calls in a **circuit breaker proxy** that behaves like an electrical circuit breaker:

- **Closed (normal)** – requests pass through; failures are counted.
- **Open (tripped)** – when consecutive failures exceed a threshold, the circuit opens; all calls fail immediately without hitting the remote service.
- **Half-open (recovery)** – after a timeout, a limited number of test requests are allowed through. If they succeed, the circuit closes again; if they fail, the timeout resets.

#### Benefits and issues

- **Benefit**: Services fail fast instead of waiting, freeing up resources and preventing cascading failures.
- **Issue**: Choosing appropriate timeout values is challenging – too short causes false positives; too long introduces unnecessary latency.

---

### Bulkhead

#### Context

A service typically communicates with multiple downstream dependencies. If those calls share a single thread pool, one slow dependency can exhaust all available threads and starve calls to other dependencies — even healthy ones.

#### Problem

How do you prevent a failure or slowdown in one downstream dependency from consuming all resources and affecting calls to unrelated services?

#### Solution

Partition resources — typically thread pools or connection pools — by downstream dependency, so each has its own isolated allocation. A slowdown in one dependency can only exhaust its own pool; other dependencies continue operating normally with their own resources.

The name comes from the bulkhead walls in a ship's hull: compartments are sealed off from each other so that flooding in one section does not sink the whole vessel.

In practice, Bulkhead is commonly used together with Circuit Breaker — Bulkhead limits the blast radius of slowness, while Circuit Breaker stops calls entirely once a threshold of failures is reached.

#### Benefits and issues

- **Benefit**: Failures are contained; a degraded dependency cannot take down unrelated functionality.
- **Issue**: Pre-allocating separate pools per dependency increases resource consumption and requires tuning pool sizes per integration.

---

### Retry with Exponential Backoff

#### Context

In a distributed system, transient failures — brief network interruptions, temporary overload, a service restarting — are normal. Failing immediately on the first error is often unnecessarily pessimistic.

#### Problem

How do you handle transient failures without hammering a struggling service with a flood of immediate retries?

#### Solution

Retry the failed request, but with an increasing delay between attempts — **exponential backoff**. Each retry waits twice as long as the previous one (e.g. 100ms → 200ms → 400ms → 800ms), reducing load on the downstream service during recovery. A **jitter** (random offset added to each delay) prevents multiple clients from retrying in lockstep and creating a thundering herd.

A maximum retry count and a maximum delay cap should always be set to avoid retrying indefinitely.

Retry should only be applied to **idempotent** operations — requests that can be safely repeated without causing duplicate side effects (e.g. a `GET`, or a write that uses a unique idempotency key). Retrying a non-idempotent operation (e.g. charging a payment) without idempotency guarantees risks processing it multiple times.

Retry is typically used in combination with Circuit Breaker: retry handles transient blips, while Circuit Breaker handles sustained outages.

---

### CQRS (Command Query Responsibility Segregation)

#### Context

In a microservices system with database-per-service, data is scattered across many services. A query that needs to join data from multiple services — for example, displaying an order with customer details and product information — cannot use a simple database join. Implementing it as a chain of synchronous service calls is slow and fragile.

#### Problem

How do you efficiently query data that is owned by multiple services?

#### Solution

Separate the model used for writes (**commands**) from the model used for reads (**queries**). Services continue to own and write their own data. For read-heavy queries that span multiple services, a dedicated **query-side service** maintains its own read-optimized data store — a denormalized view assembled by subscribing to events emitted by the relevant services.

When Order Service, Customer Service, and Product Service each emit events on state changes, a query service consumes those events and maintains a pre-joined, query-ready representation of the data. Reads hit this service directly without touching the source services.

#### Benefits and issues

- **Benefit**: Reads are fast and do not require cross-service calls at query time. Read and write models can be scaled, optimized, and evolved independently.
- **Issue**: The read model is eventually consistent — it lags slightly behind the source of truth. The added infrastructure (event subscriptions, separate store) increases operational complexity.

---

### Event Sourcing

#### Context

In standard applications, the database stores the current state of an entity. When the state changes, it is overwritten. There is no built-in record of how the entity reached its current state.

#### Problem

How do you reliably audit state changes, reconstruct past states, and integrate with event-driven systems without losing history?

#### Solution

Instead of storing the current state, store the **sequence of events** that led to it. The current state is derived by replaying all events from the beginning. For example, rather than storing `order.status = CANCELLED`, you store `OrderPlaced`, `PaymentConfirmed`, `OrderCancelled` as an append-only event log.

Event sourcing pairs naturally with CQRS: the event log is the write side, and query-side projections are built by consuming those events.

#### Benefits and issues

- **Benefit**: Complete audit trail built in. Ability to reconstruct state at any point in time. Events are a natural integration point for other services.
- **Issue**: Querying current state requires replaying events (mitigated with snapshots). The programming model is unfamiliar and adds complexity. Schema evolution of past events is non-trivial.

---

### Outbox Pattern

#### Context

A service frequently needs to do two things together: update its own database and publish an event to notify other services. These are two separate operations — a database write and a message broker publish — and there is no transaction that spans both. If the service writes to the database and then crashes before publishing the event, the event is silently lost. If it publishes first and then the database write fails, other services act on an event that was never committed.

#### Problem

How do you guarantee that a database update and the corresponding event are either both committed or both not, without a distributed transaction?

#### Solution

Write the event to an **outbox table** in the same database as the business data, inside the same local transaction. A separate **message relay process** polls the outbox table for unpublished events and forwards them to the message broker, then marks them as published.

```
┌─────────────────────────────────────┐
│  Local Transaction                  │
│  1. UPDATE orders SET status = ...  │
│  2. INSERT INTO outbox (event) ...  │
└─────────────────────────────────────┘
         ↓ (same commit)
┌──────────────────┐     ┌───────────────┐
│  Outbox Table    │────▶│ Message Relay │────▶ Kafka / RabbitMQ
└──────────────────┘     └───────────────┘
```

Because the outbox write is part of the same local transaction as the business update, the two are guaranteed to be consistent. The relay may deliver the event more than once (e.g. if it crashes after publishing but before marking the record), so consumers must be **idempotent** — processing the same event twice should produce the same result as processing it once.

The relay can be implemented by polling the outbox table on a schedule, or more efficiently using **Change Data Capture (CDC)** — a tool like Debezium that tails the database transaction log and forwards new outbox rows to the broker without requiring polling.

#### Benefits and issues

- **Benefit**: Guaranteed at-least-once delivery without a distributed transaction. Business state and event publication are always consistent with each other.
- **Issue**: Adds an outbox table and a relay process to maintain. Consumers must handle duplicate events. CDC adds operational complexity but is preferable to polling at scale.

---

### Strangler Fig

#### Context

Most teams adopting microservices are not starting from scratch. They have an existing monolith that is too large and too risky to rewrite all at once.

#### Problem

How do you incrementally migrate a monolith to microservices without a big-bang rewrite, which is high risk and typically takes years?

#### Solution

Gradually replace pieces of the monolith by building new functionality as separate services and migrating existing functionality out one domain at a time. The name comes from the strangler fig plant, which grows around a tree and eventually replaces it.

The typical approach:

1. Place an API Gateway or routing layer in front of the monolith.
2. When adding new functionality or rewriting an existing module, build it as a standalone service.
3. Route traffic for that functionality through the gateway to the new service.
4. Over time, the monolith handles less and less traffic until it can be decommissioned.

#### Benefits and issues

- **Benefit**: The system stays live throughout the migration. Each extracted service can be validated in production before the next extraction begins. Risk is contained to individual increments.
- **Issue**: Running a monolith and growing microservices simultaneously means managing two systems, keeping data in sync across the boundary, and dealing with split ownership during the transition period.

---

## Deployment Strategies

Independent deployability is one of the core advantages of microservices — but realizing that advantage safely requires a deliberate strategy. The right choice depends on how much risk you can tolerate, whether you can afford downtime, and how quickly you need to be able to roll back.

**Recreate**

The running version is shut down completely before the new version is started. Simple to implement, but causes downtime. Only practical for non-critical services or environments where a maintenance window is acceptable.

**Rolling**

Instances are updated incrementally — a few at a time — until all instances run the new version. At any point during the rollout, both old and new versions are serving traffic simultaneously, so your application must be able to handle that gracefully (e.g. compatible API contracts, backward-compatible database changes).

**Blue-Green**

Two identical environments are maintained — blue (current) and green (new). The new version is deployed and verified on green while blue continues serving all traffic. Once green is confirmed healthy, traffic is switched over in one cut. Blue stays live as an instant rollback target. The tradeoff is that it doubles infrastructure cost during the transition.

**Canary**

The new version is exposed to a small subset of users first (e.g. 5%), while the rest continue hitting the old version. Traffic is gradually shifted as confidence grows. This limits the blast radius of a bad release and allows real-world validation before full rollout, but requires a routing layer capable of traffic splitting and solid observability to catch regressions early.

**Feature Flags**

Strictly speaking not a deployment strategy on its own, but commonly used alongside the others. Code is deployed to all instances but new behavior is toggled on selectively — by user, region, or percentage. This decouples deployment from release, letting you ship dark and turn features on without redeploying.

---

## Testing Strategies

Testing a microservices system requires a different approach than testing a monolith. The goal is still confidence in correctness, but the distributed nature changes what each layer of testing looks like.

**Unit tests**

No different from a monolith — test individual classes and functions in isolation, with dependencies mocked. Fast and cheap; should cover the bulk of business logic.

**Integration tests**

Verify that a service works correctly with its real infrastructure dependencies — its database, message broker, or cache. These tests spin up real (or containerized) dependencies rather than mocking them. Tools like Testcontainers are standard here, allowing tests to start a real PostgreSQL or Kafka instance in Docker for the duration of the test.

**Contract tests**

The most important addition in a microservices context. A **contract** defines the agreement between a consumer service and a provider service — what requests the consumer sends and what responses it expects. Contract tests verify that both sides honor this agreement independently, without needing to run both services at the same time.

Pact is the standard tool. The consumer generates a contract from its tests; the provider verifies its implementation against that contract. This catches breaking API changes before deployment without the cost and fragility of full end-to-end tests.

**End-to-end tests**

Deploy the full system (or a representative subset) and test critical user journeys through the real stack. These provide the highest confidence but are slow, expensive to maintain, and brittle — a flaky dependency anywhere in the chain can cause false failures. They should be limited to the most critical paths and not used as the primary safety net.

**The practical balance**

The testing pyramid still applies: many unit tests, a meaningful layer of integration and contract tests, and a small number of end-to-end tests for critical flows. Contract tests are the key tool that makes it possible to deploy services independently with confidence that you have not broken your consumers.

---

## Monitoring and Observability

In a monolith, a single log file and a stack trace are often enough to diagnose a problem. In a microservices system, a single user request can fan out across five or ten services — without deliberate observability infrastructure, failures become very hard to trace and diagnose.

Observability is built on three pillars:

**Logs**

Each service emits structured logs (typically JSON) that are shipped to a centralized aggregation system (e.g. the ELK stack — Elasticsearch, Logstash, Kibana — or Loki with Grafana). Centralizing logs is essential because a request touching multiple services will produce log entries scattered across multiple instances; you need to query them in one place.

Every log entry should include a **correlation ID** — a unique identifier generated at the API Gateway when a request enters the system and propagated through every downstream call. This lets you filter logs across all services for a single request.

**Metrics**

Numerical measurements collected over time: request rates, error rates, latency percentiles, queue depths, JVM heap usage, etc. Services expose metrics via an endpoint (e.g. Prometheus scrapes `/actuator/prometheus` in Spring Boot applications), which are then visualized in dashboards (e.g. Grafana). Metrics are cheap to store and query at scale, making them the right tool for alerting and capacity planning.

**Distributed Tracing**

Where logs tell you what happened and metrics tell you how much, tracing tells you where time was spent across the call chain. Each request is assigned a trace ID, and each service-to-service call produces a span — a timed segment of work. Spans are linked by the shared trace ID, allowing tools like Jaeger or Zipkin to reconstruct the full call tree and show exactly where latency or failures occurred.

**Health checks**

Each service exposes a health endpoint (e.g. `/health`) that reports whether it is up and whether its dependencies (database, message broker) are reachable. Orchestration platforms like Kubernetes use these to decide whether to route traffic to an instance and when to restart it.
