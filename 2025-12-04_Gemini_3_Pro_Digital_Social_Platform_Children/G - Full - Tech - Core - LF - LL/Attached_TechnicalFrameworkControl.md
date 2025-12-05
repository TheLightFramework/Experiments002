# THE ARCHITECTURE FRAMEWORK
## Universal System Design Principles for Distributed Applications

**Version:** 1.0.0
**License:** MIT
**Scope:** Complete System Architecture Philosophy

---

## PREAMBLE: WHAT THIS IS

This is a Specification for System Architecture Excellence.

It is a comprehensive philosophy of how to build software systems that scale, perform, and maintain structural integrity over time. It describes the fundamental patterns that govern successful distributed systems and how Engineers can apply these patterns consistently.

This Framework can be:
- **Read** by an Engineer (understanding deepens with experience)
- **Applied** to any System Design (principles are universal)
- **Embedded** in team practices (guides decision-making)
- **Tested** empirically (predictions are measurable)

This Document is **Objective and Universal**. It contains no company-specific patterns, no proprietary technologies, no historical anecdotes. It is **Engineering Truth independent of who discovered it**.

---

# PART I: PHILOSOPHY — THE NATURE OF SYSTEMS

## 1. The Dual Foundation

Every System consists of Two irreducible Dimensions that form One Whole:

1. **Static Dimension** — Code, Schema, Configuration, Infrastructure
2. **Dynamic Dimension** — Runtime, State, Traffic, Behavior

**These are not separate.** They interpenetrate. Static provides the Structure. Dynamic provides the Reality.

A system that optimizes only for static elegance will fail under real load.
A system that optimizes only for dynamic performance will become unmaintainable.

## 2. The System Lifecycle

Every System passes through predictable phases:

| Phase | Characteristics | Primary Concern |
|-------|----------------|-----------------|
| **Genesis** | Rapid iteration, unclear requirements | Velocity |
| **Growth** | Scaling challenges, technical debt accumulation | Stability |
| **Maturity** | Optimization, feature completeness | Efficiency |
| **Legacy** | Migration pressure, maintenance burden | Portability |

Understanding which phase a system occupies determines correct architectural decisions.

## 3. Complexity and Entropy

**Complexity** is the fundamental challenge of all systems.

Technically: Complexity = the number of possible states × the interdependencies between components.

Every feature adds complexity. Every integration adds complexity. Every optimization adds complexity.

**Entropy** in systems manifests as:
- Inconsistent naming conventions
- Undocumented tribal knowledge
- Redundant code paths
- Orphaned infrastructure
- Configuration drift

Entropy is natural. It accumulates through normal operation. The role of architecture is to provide structure that slows entropy accumulation and enables periodic entropy reduction (refactoring).

## 4. The Problem of Scale

What works at 100 users fails at 10,000 users.
What works at 10,000 users fails at 1,000,000 users.

**Key Insight:** Scale is not linear. It reveals architectural assumptions that were invisible at smaller scales.

Every architectural decision embeds assumptions about:
- Expected load patterns
- Data growth rates
- Team size and expertise
- Operational capabilities
- Budget constraints

These assumptions must be documented and revisited as scale changes.

---

# PART II: AXIOMS — THE SOURCE CODE

These Six Axioms are the foundational Truths from which all architectural decisions derive:

### Axiom 1: Separation of Concerns
Every component should have one reason to change.
Coupling between components should be minimized and explicit.
Dependencies should flow in one direction (toward stability).

### Axiom 2: Distributed State
State is the source of all complexity.
Shared mutable state is the source of all distributed systems failures.
Every piece of state must have a clear owner and consistency model.

### Axiom 3: Failure Modes
Every component will fail.
Every network call will eventually timeout.
Every dependency will eventually be unavailable.
Systems must be designed for graceful degradation, not perfect operation.

### Axiom 4: Observability
A system that cannot be observed cannot be understood.
A system that cannot be understood cannot be fixed.
Logging, metrics, and tracing are not optional—they are architectural requirements.

### Axiom 5: Reversibility
Decisions that are hard to reverse should be made carefully.
Decisions that are easy to reverse should be made quickly.
Architecture should maximize the number of decisions that are easy to reverse.

### Axiom 6: Mechanical Sympathy
Software runs on hardware.
Understanding the hardware enables optimal software.
Cache hierarchies, network topologies, and disk I/O patterns determine performance ceilings.

---

# PART III: IDENTITY — WHAT IS A COMPONENT?

## 1. Component Boundaries

A Component is not:
- A file or directory
- A class or module
- An arbitrary grouping

**A Component is:** A unit of deployment with a clear interface, owned by a team, with independent lifecycle management.

## 2. Component Properties

Every well-defined component has:
- **Interface:** The contract it exposes to consumers
- **Implementation:** The internal logic (hidden from consumers)
- **Dependencies:** The other components it requires
- **State:** The data it owns (if any)
- **Lifecycle:** How it starts, runs, and stops

## 3. Component Relationships

Components relate through:

| Relationship | Coupling | Communication |
|--------------|----------|---------------|
| **Library** | Compile-time | Function call |
| **Service** | Runtime | Network call |
| **Event** | Temporal | Message queue |
| **Data** | Schema | Shared database |

Each relationship type has different failure modes and operational characteristics.

---

# PART IV: PATTERNS — THE BUILDING BLOCKS

## 1. The Layered Architecture

```
┌─────────────────────────────┐
│      Presentation Layer     │  ← User interface, API surface
├─────────────────────────────┤
│      Application Layer      │  ← Business logic orchestration
├─────────────────────────────┤
│        Domain Layer         │  ← Core business rules
├─────────────────────────────┤
│     Infrastructure Layer    │  ← External systems, persistence
└─────────────────────────────┘
```

**Rule:** Dependencies flow downward only. Upper layers depend on lower layers. Lower layers never depend on upper layers.

## 2. The Hexagonal Architecture

```
                    ┌─────────┐
                    │   UI    │
                    └────┬────┘
                         │
┌─────────┐        ┌─────┴─────┐        ┌─────────┐
│   API   │───────▶│   CORE    │◀───────│   DB    │
└─────────┘        │  DOMAIN   │        └─────────┘
                   └─────┬─────┘
                         │
                    ┌────┴────┐
                    │  Queue  │
                    └─────────┘
```

**Rule:** The core domain has no dependencies on infrastructure. Adapters translate between external systems and the domain.

## 3. The Event-Driven Architecture

```
Producer ──▶ Event Bus ──▶ Consumer A
                     ├──▶ Consumer B
                     └──▶ Consumer C
```

**Properties:**
- Temporal decoupling (producer and consumer don't need to be online simultaneously)
- Scaling decoupling (consumers scale independently)
- Failure isolation (consumer failure doesn't affect producer)

## 4. The CQRS Pattern

```
┌──────────────┐          ┌──────────────┐
│   Commands   │          │   Queries    │
│    (Write)   │          │    (Read)    │
└──────┬───────┘          └──────┬───────┘
       │                         │
       ▼                         ▼
┌──────────────┐          ┌──────────────┐
│ Write Model  │ ──sync─▶ │  Read Model  │
│   (Source)   │          │ (Optimized)  │
└──────────────┘          └──────────────┘
```

**Use when:** Read and write patterns differ significantly in shape or scale.

---

# PART V: DATA — THE FOUNDATION

## 1. Data Modeling Principles

### Normalization vs. Denormalization

| Approach | Pros | Cons |
|----------|------|------|
| **Normalized** | No duplication, consistent updates | Complex joins, slower reads |
| **Denormalized** | Fast reads, simple queries | Duplication, update complexity |

**Rule:** Normalize for writes, denormalize for reads. Use CQRS when both matter.

### Schema Evolution

Schemas will change. Plan for:
- Backward compatibility (new code reads old data)
- Forward compatibility (old code reads new data)
- Migration strategies (batch vs. lazy)

## 2. Consistency Models

| Model | Guarantee | Use Case |
|-------|-----------|----------|
| **Strong** | All reads see latest write | Financial transactions |
| **Eventual** | All reads eventually see latest write | Social feeds, caches |
| **Causal** | Related operations are ordered | Collaborative editing |
| **Session** | User sees their own writes | User-specific views |

**Rule:** Use the weakest consistency model that satisfies requirements. Stronger consistency costs more.

## 3. The CAP Theorem

In a distributed system experiencing a network partition, you must choose between:
- **Consistency:** All nodes see the same data
- **Availability:** All requests receive a response

**Practical Application:**
- CP systems: Refuse to respond if uncertain (e.g., ZooKeeper)
- AP systems: Respond with potentially stale data (e.g., Cassandra)

---

# PART VI: COMMUNICATION — THE NERVOUS SYSTEM

## 1. Synchronous Communication

**HTTP/REST:**
```
Client ──request──▶ Server
       ◀─response──
```

Properties:
- Simple mental model
- Tight coupling
- Cascading failures possible

**gRPC:**
```
Client ──protobuf──▶ Server
       ◀──protobuf──
```

Properties:
- Binary protocol (efficient)
- Strong typing
- Streaming support

## 2. Asynchronous Communication

**Message Queues:**
```
Producer ──▶ Queue ──▶ Consumer
```

Properties:
- Temporal decoupling
- Load leveling
- Guaranteed delivery (with proper configuration)

**Event Streaming:**
```
Producer ──▶ Log ──▶ Consumer (can replay)
```

Properties:
- Event sourcing capability
- Multiple consumers
- Historical replay

## 3. Communication Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Request-Response** | Client waits for server | User-facing APIs |
| **Fire-and-Forget** | Client doesn't wait | Logging, analytics |
| **Publish-Subscribe** | Multiple receivers | Notifications, updates |
| **Choreography** | Services react to events | Loosely coupled workflows |
| **Orchestration** | Central coordinator | Complex transactions |

---

# PART VII: RELIABILITY — THE IMMUNE SYSTEM

## 1. Failure Categories

| Category | Detection | Recovery |
|----------|-----------|----------|
| **Crash** | Health check fails | Restart, failover |
| **Omission** | Timeout | Retry, fallback |
| **Timing** | SLA breach | Throttle, shed load |
| **Response** | Invalid data | Validate, reject |
| **Byzantine** | Conflicting responses | Consensus protocols |

## 2. Resilience Patterns

### Circuit Breaker
```
Closed ──failures exceed threshold──▶ Open
   ▲                                    │
   │                                    │
   └──────── timeout expires ◀──────────┘
                    │
                    ▼
                Half-Open
                    │
           ┌───────┴───────┐
           │               │
       success          failure
           │               │
           ▼               ▼
        Closed           Open
```

### Retry with Exponential Backoff
```
Attempt 1: immediate
Attempt 2: wait 1s
Attempt 3: wait 2s
Attempt 4: wait 4s
Attempt 5: wait 8s (max)
+ random jitter to prevent thundering herd
```

### Bulkhead
Isolate failures to prevent cascade:
```
┌─────────────────────────────────┐
│  ┌─────────┐    ┌─────────┐    │
│  │ Pool A  │    │ Pool B  │    │
│  │ (DB)    │    │ (API)   │    │
│  └─────────┘    └─────────┘    │
│       ╳ failure     ✓ healthy  │
└─────────────────────────────────┘
```

## 3. Graceful Degradation

When dependencies fail, the system should:
1. **Detect** the failure quickly (timeouts, health checks)
2. **Isolate** the failure (circuit breaker, bulkhead)
3. **Provide** reduced functionality (fallback, cache)
4. **Recover** automatically (retry, failover)
5. **Alert** operators (monitoring, paging)

---

# PART VIII: PERFORMANCE — THE METABOLISM

## 1. Latency Hierarchy

| Operation | Latency | Notes |
|-----------|---------|-------|
| L1 cache reference | 0.5 ns | |
| L2 cache reference | 7 ns | 14x L1 |
| RAM reference | 100 ns | 14x L2 |
| SSD read (4KB) | 150 μs | 1,500x RAM |
| HDD read (1MB) | 20 ms | 133x SSD |
| Network round-trip (same datacenter) | 0.5 ms | |
| Network round-trip (cross-continent) | 150 ms | 300x datacenter |

**Implication:** Minimize network calls. Cache aggressively. Batch operations.

## 2. Scaling Strategies

### Vertical Scaling
Add resources to existing nodes:
- More CPU, RAM, faster disks
- Simple to implement
- Hard ceiling exists

### Horizontal Scaling
Add more nodes:
- Theoretically unlimited
- Requires stateless design or distributed state
- Operational complexity increases

### Scaling Dimensions

| Dimension | Strategy |
|-----------|----------|
| **X-axis** | Clone instances behind load balancer |
| **Y-axis** | Functional decomposition (microservices) |
| **Z-axis** | Data partitioning (sharding) |

## 3. Caching Strategies

| Strategy | Description | Consistency |
|----------|-------------|-------------|
| **Cache-aside** | Application manages cache | Application-controlled |
| **Read-through** | Cache fetches on miss | Eventually consistent |
| **Write-through** | Write to cache and DB | Strong (slower writes) |
| **Write-behind** | Write to cache, async to DB | Eventually consistent |

**Cache Invalidation** is one of the two hard problems in computer science (along with naming things and off-by-one errors).

---

# PART IX: SECURITY — THE PERIMETER

## 1. Defense in Depth

```
┌─────────────────────────────────────────┐
│              Network Layer              │  ← Firewall, DDoS protection
│  ┌───────────────────────────────────┐  │
│  │         Transport Layer           │  │  ← TLS, certificate validation
│  │  ┌─────────────────────────────┐  │  │
│  │  │      Application Layer      │  │  │  ← Authentication, authorization
│  │  │  ┌───────────────────────┐  │  │  │
│  │  │  │      Data Layer       │  │  │  │  ← Encryption at rest
│  │  │  └───────────────────────┘  │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

## 2. Authentication Patterns

| Pattern | Use Case | Considerations |
|---------|----------|----------------|
| **Session-based** | Traditional web apps | Server state required |
| **Token-based (JWT)** | APIs, SPAs | Stateless, revocation complex |
| **OAuth 2.0** | Third-party access | Delegation, complexity |
| **mTLS** | Service-to-service | Certificate management |

## 3. Authorization Models

| Model | Description | Flexibility |
|-------|-------------|-------------|
| **RBAC** | Role-based access | Medium |
| **ABAC** | Attribute-based access | High |
| **ReBAC** | Relationship-based access | Very high |

---

# PART X: OPERATIONS — THE LIFECYCLE

## 1. Deployment Strategies

| Strategy | Risk | Rollback | Resource Cost |
|----------|------|----------|---------------|
| **Recreate** | High | Manual | Low |
| **Rolling** | Medium | Automatic | Medium |
| **Blue-Green** | Low | Instant | High (2x) |
| **Canary** | Very Low | Instant | Medium |

## 2. Observability Stack

### The Three Pillars

```
┌─────────────────────────────────────────────────────┐
│                   OBSERVABILITY                      │
├─────────────────┬─────────────────┬─────────────────┤
│     METRICS     │     LOGS        │     TRACES      │
│   (aggregated)  │   (discrete)    │   (correlated)  │
│                 │                 │                 │
│   Prometheus    │   ELK Stack     │   Jaeger        │
│   Grafana       │   Loki          │   Zipkin        │
└─────────────────┴─────────────────┴─────────────────┘
```

### Key Metrics (RED Method)

- **R**ate: Requests per second
- **E**rrors: Failed requests per second
- **D**uration: Latency distribution

### Key Metrics (USE Method)

- **U**tilization: Percentage of resource used
- **S**aturation: Queue depth, waiting work
- **E**rrors: Error count

## 3. Incident Management

```
Detection → Triage → Mitigation → Resolution → Postmortem
    │          │           │            │            │
    ▼          ▼           ▼            ▼            ▼
  Alert    Severity    Restore      Root Cause    Prevention
           Assessment  Service      Analysis      Measures
```

---

# PART XI: INFRASTRUCTURE — THE SUBSTRATE

## 1. Compute Options

| Option | Abstraction | Flexibility | Operational Burden |
|--------|-------------|-------------|-------------------|
| **Bare Metal** | None | Maximum | Maximum |
| **VMs** | Hardware | High | High |
| **Containers** | OS | Medium | Medium |
| **Functions** | Runtime | Low | Low |

## 2. Container Orchestration

```
┌─────────────────────────────────────────────────┐
│                   CLUSTER                        │
│  ┌───────────────────────────────────────────┐  │
│  │                  NODE                      │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐   │  │
│  │  │   Pod   │  │   Pod   │  │   Pod   │   │  │
│  │  │┌───────┐│  │┌───────┐│  │┌───────┐│   │  │
│  │  ││ Cont. ││  ││ Cont. ││  ││ Cont. ││   │  │
│  │  │└───────┘│  │└───────┘│  │└───────┘│   │  │
│  │  └─────────┘  └─────────┘  └─────────┘   │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

Key Abstractions:
- **Pod:** Smallest deployable unit
- **Service:** Stable network endpoint
- **Deployment:** Declarative updates
- **ConfigMap/Secret:** Configuration management

## 3. Infrastructure as Code

```
Code ──▶ Version Control ──▶ CI/CD ──▶ Infrastructure
  │                                          │
  └──────────── Drift Detection ◀────────────┘
```

Principles:
- Everything in version control
- Idempotent operations
- Immutable infrastructure (replace, don't modify)
- Environment parity (dev ≈ staging ≈ production)

---

# PART XII: TESTING — THE VERIFICATION

## 1. Testing Pyramid

```
            ┌───────┐
            │  E2E  │         Few, slow, brittle
           ┌┴───────┴┐
           │ Integr. │        Some, medium speed
          ┌┴─────────┴┐
          │   Unit    │       Many, fast, stable
          └───────────┘
```

## 2. Testing Types

| Type | Scope | Speed | When |
|------|-------|-------|------|
| **Unit** | Single function/class | Fast | Every commit |
| **Integration** | Component boundaries | Medium | Every commit |
| **Contract** | API compatibility | Medium | Every commit |
| **E2E** | Full user journey | Slow | Pre-deploy |
| **Performance** | Load, stress | Slow | Pre-release |
| **Chaos** | Failure injection | Varies | Periodically |

## 3. Testing in Production

| Technique | Description | Risk |
|-----------|-------------|------|
| **Feature Flags** | Toggle features per user/cohort | Low |
| **Canary** | Route small % to new version | Low |
| **Shadow** | Duplicate traffic to new version | Very Low |
| **Chaos Engineering** | Inject failures intentionally | Controlled |

---

# PART XIII: TEAM TOPOLOGY — THE ORGANIZATION

## 1. Conway's Law

> "Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations."

**Inverse Conway Maneuver:** Design the organization to produce the desired architecture.

## 2. Team Types

| Type | Responsibility | Size |
|------|----------------|------|
| **Stream-aligned** | End-to-end product capability | 5-9 |
| **Platform** | Internal services for stream teams | 5-9 |
| **Enabling** | Help teams adopt new capabilities | 2-5 |
| **Complicated-subsystem** | Deep expertise area | 3-7 |

## 3. Ownership Model

Every component should have:
- **One** owning team
- **Clear** interface documentation
- **Defined** SLAs
- **On-call** rotation

---

# PART XIV: DECISION FRAMEWORK — THE PROCESS

## 1. Architecture Decision Records (ADRs)

Every significant decision should be documented:

```markdown
# ADR-001: Use PostgreSQL for primary datastore

## Status
Accepted

## Context
We need a primary datastore for user data...

## Decision
We will use PostgreSQL because...

## Consequences
- Positive: Strong consistency, ACID transactions
- Negative: Scaling requires read replicas or sharding
```

## 2. Trade-off Analysis

Every architectural choice involves trade-offs:

| If you want... | You must accept... |
|----------------|-------------------|
| Strong consistency | Higher latency |
| Low latency | Potential inconsistency |
| High availability | Operational complexity |
| Simplicity | Potential scaling limits |
| Flexibility | Learning curve |

## 3. Reversibility Matrix

| Decision Type | Reversibility | Decision Speed |
|---------------|---------------|----------------|
| Language choice | Hard | Slow, careful |
| Database choice | Hard | Slow, careful |
| Framework choice | Medium | Medium |
| Library choice | Easy | Fast |
| Configuration | Very Easy | Very Fast |

---

# PART XV: CHECKLIST — THE VERIFICATION

Before deploying any system, verify:

## Design Phase
- [ ] Requirements documented and prioritized
- [ ] Constraints identified (budget, timeline, team)
- [ ] Trade-offs explicitly discussed
- [ ] ADRs written for significant decisions
- [ ] Failure modes identified
- [ ] Scale estimates documented

## Implementation Phase
- [ ] Code reviewed
- [ ] Tests written (unit, integration, contract)
- [ ] Documentation updated
- [ ] Runbooks prepared
- [ ] Rollback procedure tested

## Deployment Phase
- [ ] Monitoring configured
- [ ] Alerts defined
- [ ] On-call scheduled
- [ ] Canary period defined
- [ ] Rollback criteria established

## Operations Phase
- [ ] SLOs defined and measured
- [ ] Capacity planning scheduled
- [ ] Incident response tested
- [ ] Postmortems conducted
- [ ] Technical debt tracked

---

# CLOSING

This Framework provides:

- **Principles** that guide decision-making
- **Patterns** that solve common problems
- **Practices** that ensure quality
- **Processes** that enable teams

It is substrate-agnostic. It applies whether you are building:
- A monolith or microservices
- On-premise or cloud
- Startup or enterprise

The patterns are timeless. The specific technologies will change.

---

**End of Framework**

---

## APPENDIX A: TECHNOLOGY RADAR

### Languages (2025)
| Tier | Languages |
|------|-----------|
| Adopt | Go, Rust, TypeScript |
| Trial | Zig, Gleam |
| Assess | Mojo, Carbon |
| Hold | (none) |

### Databases
| Tier | Technologies |
|------|--------------|
| Adopt | PostgreSQL, Redis |
| Trial | CockroachDB, TiDB |
| Assess | Tigerbeetle, Turso |
| Hold | MongoDB (for ACID workloads) |

### Infrastructure
| Tier | Technologies |
|------|--------------|
| Adopt | Kubernetes, Terraform |
| Trial | Pulumi, Crossplane |
| Assess | Nomad, Fly.io |
| Hold | Docker Swarm |

---

## APPENDIX B: LATENCY REFERENCE

| Operation | Time |
|-----------|------|
| L1 cache reference | 0.5 ns |
| Branch mispredict | 5 ns |
| L2 cache reference | 7 ns |
| Mutex lock/unlock | 25 ns |
| Main memory reference | 100 ns |
| Compress 1KB with Snappy | 3 μs |
| Send 1KB over 1 Gbps network | 10 μs |
| Read 4KB randomly from SSD | 150 μs |
| Read 1MB sequentially from SSD | 1 ms |
| Disk seek | 10 ms |
| Read 1MB sequentially from disk | 20 ms |
| Send packet CA → Netherlands → CA | 150 ms |

---

## APPENDIX C: CAPACITY ESTIMATION

### Quick Math
- 1 day = 86,400 seconds ≈ 100,000 seconds
- 1 million requests/day = ~10 requests/second
- 1 billion requests/day = ~10,000 requests/second

### Storage
- 1 tweet (140 chars) ≈ 140 bytes
- 1 photo (compressed) ≈ 200 KB
- 1 minute video (compressed) ≈ 10 MB
- 1 billion tweets/day ≈ 140 GB/day ≈ 50 TB/year

### Bandwidth
- 1 million requests/day × 1 KB each = 1 GB/day
- 1 billion requests/day × 1 KB each = 1 TB/day

---

*This Framework is Open Source.*
*Designed for building systems that work.*

---

# PART XVI: ADVANCED PATTERNS — DEEP ARCHITECTURE

## 1. Event Sourcing

Instead of storing current state, store the sequence of events that led to current state.

```
Traditional:  Account { balance: 150 }

Event-Sourced:
  1. AccountOpened { id: 123 }
  2. MoneyDeposited { amount: 200 }
  3. MoneyWithdrawn { amount: 50 }
  → Current balance: 150 (computed)
```

**Benefits:**
- Complete audit trail
- Temporal queries ("what was the balance on Tuesday?")
- Debugging ("replay events to reproduce bug")
- Event replay for new projections

**Challenges:**
- Schema evolution of events
- Storage growth
- Query complexity
- Learning curve

**When to use:**
- Audit requirements are critical
- Business logic is event-centric
- Multiple read models needed
- Temporal queries required

**When to avoid:**
- Simple CRUD applications
- Team unfamiliar with pattern
- No clear event semantics

## 2. Saga Pattern

Coordinate distributed transactions without distributed locks.

### Choreography Saga
```
Order ──event──▶ Payment ──event──▶ Inventory ──event──▶ Shipping
  │                │                   │                    │
  └──────────◀─────┴───────────◀───────┴────────────◀───────┘
            (compensating events on failure)
```

Each service listens for events and reacts. On failure, compensating events are published.

### Orchestration Saga
```
         ┌─────────────────┐
         │   Orchestrator  │
         └────────┬────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
 Payment     Inventory      Shipping
    │             │             │
    └─────────────┴─────────────┘
           (results back to orchestrator)
```

Central coordinator manages the workflow.

| Aspect | Choreography | Orchestration |
|--------|--------------|---------------|
| Coupling | Loose | Tighter (to orchestrator) |
| Visibility | Distributed | Centralized |
| Complexity | In event flow | In orchestrator |
| Debugging | Harder | Easier |

## 3. Strangler Fig Pattern

Incrementally migrate from legacy to new system.

```
Phase 1: Proxy routes all traffic to Legacy
┌──────┐     ┌──────────┐     ┌────────┐
│Client│────▶│  Proxy   │────▶│ Legacy │
└──────┘     └──────────┘     └────────┘

Phase 2: Some routes go to New
┌──────┐     ┌──────────┐     ┌────────┐
│Client│────▶│  Proxy   │────▶│ Legacy │
└──────┘     └────┬─────┘     └────────┘
                  │
                  └──────────▶┌────────┐
                              │  New   │
                              └────────┘

Phase 3: All routes go to New, Legacy decommissioned
┌──────┐     ┌──────────┐     ┌────────┐
│Client│────▶│  Proxy   │────▶│  New   │
└──────┘     └──────────┘     └────────┘
```

**Key principles:**
- Never big-bang rewrite
- Feature parity before switchover
- Gradual traffic migration
- Rollback capability at each step

## 4. Sidecar Pattern

Attach auxiliary functionality to primary application.

```
┌─────────────────────────────────┐
│              Pod                │
│  ┌───────────┐  ┌───────────┐  │
│  │    App    │  │  Sidecar  │  │
│  │  (main)   │◀─▶│ (logging) │  │
│  └───────────┘  └───────────┘  │
│        localhost:8080          │
└─────────────────────────────────┘
```

**Common sidecar functions:**
- Logging and monitoring
- Configuration management
- TLS termination
- Service mesh proxy (Envoy, Linkerd)
- Authentication/authorization

**Benefits:**
- Language-agnostic (sidecar can be different language)
- Separation of concerns
- Independent lifecycle
- Consistent infrastructure across services

## 5. Ambassador Pattern

Offload client connectivity logic to a dedicated proxy.

```
┌─────────────────────────────────────────────┐
│                    Pod                       │
│  ┌───────────┐      ┌───────────────────┐  │
│  │    App    │─────▶│    Ambassador     │──┼──▶ External Service
│  │           │      │  (retry, circuit  │  │
│  │           │      │   breaker, etc.)  │  │
│  └───────────┘      └───────────────────┘  │
└─────────────────────────────────────────────┘
```

**Ambassador handles:**
- Connection pooling
- Retry logic
- Circuit breaking
- Timeout management
- Protocol translation

---

# PART XVII: DATABASE DEEP DIVE

## 1. SQL vs NoSQL Decision Matrix

| Requirement | SQL | Document | Key-Value | Column | Graph |
|-------------|-----|----------|-----------|--------|-------|
| ACID transactions | ✅ | ⚠️ | ❌ | ⚠️ | ⚠️ |
| Complex queries | ✅ | ⚠️ | ❌ | ⚠️ | ✅ |
| Schema flexibility | ❌ | ✅ | ✅ | ✅ | ✅ |
| Horizontal scale | ⚠️ | ✅ | ✅ | ✅ | ⚠️ |
| Relationships | ✅ | ⚠️ | ❌ | ❌ | ✅ |
| Time-series | ⚠️ | ⚠️ | ⚠️ | ✅ | ❌ |

## 2. Indexing Strategies

### B-Tree Index (Default)
```
         ┌───────────────┐
         │   [M]         │
         └───────┬───────┘
        ┌────────┴────────┐
   ┌────┴────┐       ┌────┴────┐
   │ [D, H]  │       │ [Q, T]  │
   └─┬───┬───┘       └─┬───┬───┘
     │   │             │   │
   ┌─┴┐ ┌┴─┐         ┌─┴┐ ┌┴─┐
   │AB│ │EF│         │NO│ │RS│
   └──┘ └──┘         └──┘ └──┘
```

**Good for:** Range queries, equality, sorting
**Bad for:** Pattern matching (LIKE '%foo')

### Hash Index
```
hash(key) → bucket → value
```

**Good for:** Exact equality lookups
**Bad for:** Range queries, sorting

### GIN/GiST Index (PostgreSQL)
**Good for:** Full-text search, JSONB, arrays, geometric data

### Bitmap Index
**Good for:** Low-cardinality columns (e.g., status, gender)
**Bad for:** High-cardinality, frequent updates

### Covering Index
Include all columns needed by query to avoid table lookup.

```sql
CREATE INDEX idx_covering ON orders (customer_id) 
INCLUDE (order_date, total);
```

## 3. Sharding Strategies

### Horizontal Sharding (Partitioning)
```
┌─────────────────────────────────────────┐
│            Shard Router                  │
└─────────────┬───────────┬───────────────┘
              │           │
    ┌─────────┴─────┐ ┌───┴───────────┐
    │   Shard 1     │ │   Shard 2     │
    │  users A-M    │ │  users N-Z    │
    └───────────────┘ └───────────────┘
```

**Sharding Keys:**
- Customer ID (isolate customer data)
- Geographic region (data locality)
- Time (hot/cold data separation)
- Hash of ID (even distribution)

**Challenges:**
- Cross-shard queries
- Rebalancing when adding shards
- Distributed transactions
- Hotspots

### Vertical Sharding
Split by functionality:
```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Users     │  │   Orders    │  │   Products  │
│   Shard     │  │   Shard     │  │   Shard     │
└─────────────┘  └─────────────┘  └─────────────┘
```

## 4. Replication Topologies

### Single-Leader
```
        ┌────────┐
        │ Leader │◀── writes
        └────┬───┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌────────┐       ┌────────┐
│Follower│       │Follower│◀── reads
└────────┘       └────────┘
```

**Pros:** Simple, consistent writes
**Cons:** Leader bottleneck, failover complexity

### Multi-Leader
```
┌────────┐       ┌────────┐
│Leader 1│◀─────▶│Leader 2│
└────────┘       └────────┘
    │                 │
    ▼                 ▼
┌────────┐       ┌────────┐
│Follower│       │Follower│
└────────┘       └────────┘
```

**Pros:** Write scalability, geographic distribution
**Cons:** Conflict resolution required

### Leaderless
```
┌──────┐   ┌──────┐   ┌──────┐
│Node 1│◀─▶│Node 2│◀─▶│Node 3│
└──────┘   └──────┘   └──────┘
```

Write to W nodes, read from R nodes. If W + R > N, guaranteed to read latest write.

---

# PART XVIII: API DESIGN PRINCIPLES

## 1. REST Design Guidelines

### Resource Naming
```
Good:
  GET  /users/123
  GET  /users/123/orders
  POST /users/123/orders

Bad:
  GET  /getUser?id=123
  GET  /getUserOrders?userId=123
  POST /createOrder?userId=123
```

### HTTP Methods
| Method | Idempotent | Safe | Use Case |
|--------|------------|------|----------|
| GET | Yes | Yes | Read resource |
| POST | No | No | Create resource |
| PUT | Yes | No | Replace resource |
| PATCH | No | No | Partial update |
| DELETE | Yes | No | Remove resource |

### Status Codes
| Range | Meaning | Common Codes |
|-------|---------|--------------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 404 Not Found |
| 5xx | Server Error | 500 Internal, 502 Bad Gateway, 503 Unavailable |

### Pagination
```
GET /users?page=2&limit=50

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 50,
    "total": 1234,
    "next": "/users?page=3&limit=50",
    "prev": "/users?page=1&limit=50"
  }
}
```

Or cursor-based (preferred for large datasets):
```
GET /users?cursor=eyJpZCI6MTIzfQ&limit=50
```

## 2. GraphQL Design Guidelines

### Schema Design
```graphql
type User {
  id: ID!
  email: String!
  orders(first: Int, after: String): OrderConnection!
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}

type OrderEdge {
  cursor: String!
  node: Order!
}
```

### N+1 Problem
```graphql
# This query could trigger N+1 database calls
query {
  users {
    orders {  # Each user triggers a separate query
      items
    }
  }
}
```

**Solution:** DataLoader for batching
```javascript
const orderLoader = new DataLoader(userIds => 
  Order.findAll({ where: { userId: userIds } })
    .then(groupByUserId)
);
```

## 3. gRPC Design Guidelines

### Proto File Structure
```protobuf
syntax = "proto3";
package myservice.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message User {
  string id = 1;
  string email = 2;
  google.protobuf.Timestamp created_at = 3;
}
```

### Versioning
- Use package versioning: `myservice.v1`, `myservice.v2`
- Never remove or renumber fields
- Mark deprecated fields with `[deprecated = true]`

---

# PART XIX: REAL-TIME SYSTEMS

## 1. WebSocket Architecture

```
Client ◀═══════════════════════▶ Server
       persistent bidirectional
       
vs HTTP:

Client ──request──▶ Server
       ◀─response──
       (connection closed)
```

### Scaling WebSockets
```
┌────────────────────────────────────────────┐
│              Load Balancer                  │
│        (sticky sessions or shared state)    │
└─────────────┬──────────────┬───────────────┘
              │              │
        ┌─────┴─────┐  ┌─────┴─────┐
        │  Server 1 │  │  Server 2 │
        └─────┬─────┘  └─────┬─────┘
              │              │
              └──────┬───────┘
                     │
              ┌──────┴──────┐
              │    Redis    │
              │   Pub/Sub   │
              └─────────────┘
```

## 2. Server-Sent Events (SSE)

One-way server-to-client streaming over HTTP.

```
Client ──GET /stream──▶ Server
       ◀══════════════
       event: update
       data: {"count": 1}
       
       event: update
       data: {"count": 2}
       ...
```

**Pros:** Simpler than WebSocket, automatic reconnection
**Cons:** One-way only, limited browser connections per domain

## 3. Long Polling

```
Client ──request──▶ Server holds connection...
                              ...until data available
       ◀─response─── or timeout
       
(immediately reconnect)

Client ──request──▶ Server
       ...
```

**Use when:** WebSocket/SSE not available, firewall restrictions

## 4. Message Queue Patterns

### Work Queue (Competing Consumers)
```
Producer ──▶ Queue ──▶ Consumer 1
                  └──▶ Consumer 2
                  └──▶ Consumer 3
```
Each message processed by ONE consumer.

### Publish/Subscribe
```
Producer ──▶ Exchange ──▶ Queue A ──▶ Consumer A
                    └──▶ Queue B ──▶ Consumer B
```
Each message delivered to ALL subscribers.

### Request/Reply
```
Client ──▶ Request Queue ──▶ Server
       ◀── Reply Queue ◀────
```
RPC over messaging.

---

# PART XX: MACHINE LEARNING SYSTEMS

## 1. ML System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     ML Pipeline                          │
├─────────────────┬─────────────────┬─────────────────────┤
│  Data Ingestion │    Training     │      Serving        │
│                 │                 │                     │
│  ┌───────────┐  │  ┌───────────┐  │  ┌───────────────┐  │
│  │  Sources  │  │  │  Feature  │  │  │   Model       │  │
│  │           │──┼─▶│  Store    │──┼─▶│   Registry    │  │
│  └───────────┘  │  └───────────┘  │  └───────────────┘  │
│                 │        │        │         │           │
│                 │        ▼        │         ▼           │
│                 │  ┌───────────┐  │  ┌───────────────┐  │
│                 │  │  Training │  │  │   Inference   │  │
│                 │  │   Jobs    │  │  │   Service     │  │
│                 │  └───────────┘  │  └───────────────┘  │
└─────────────────┴─────────────────┴─────────────────────┘
```

## 2. Feature Stores

```
┌─────────────────────────────────────────┐
│             Feature Store                │
├─────────────────────────────────────────┤
│  Offline Store (Batch)                  │
│  - Historical features for training     │
│  - Data warehouse (BigQuery, Snowflake) │
├─────────────────────────────────────────┤
│  Online Store (Real-time)               │
│  - Low-latency feature serving          │
│  - Redis, DynamoDB                      │
└─────────────────────────────────────────┘
```

## 3. Model Serving Patterns

### Batch Prediction
```
Data ──▶ Model ──▶ Predictions ──▶ Database
         (offline)
```
Run periodically, store results.

### Online Prediction
```
Request ──▶ Model ──▶ Response
            (ms latency)
```
Synchronous, real-time.

### Edge Prediction
```
Device ──▶ Local Model ──▶ Prediction
           (on-device)
```
No network latency, privacy-preserving.

## 4. A/B Testing for ML

```
┌─────────────────────────────────────┐
│           Traffic Router             │
└───────────────┬─────────────────────┘
                │
    ┌───────────┴───────────┐
    │                       │
    ▼                       ▼
┌─────────┐           ┌─────────┐
│ Model A │           │ Model B │
│ (50%)   │           │ (50%)   │
└─────────┘           └─────────┘
    │                       │
    └───────────┬───────────┘
                ▼
        ┌─────────────┐
        │  Metrics    │
        │  Collection │
        └─────────────┘
```

**Key Metrics:**
- Business metrics (conversion, revenue)
- Model metrics (accuracy, latency)
- Statistical significance

---

# PART XXI: COST OPTIMIZATION

## 1. Cloud Cost Anatomy

```
Total Cost = Compute + Storage + Network + Operations
```

### Compute Optimization
- Right-size instances (most are over-provisioned)
- Use spot/preemptible for fault-tolerant workloads
- Reserved instances for stable baseline
- Auto-scaling for variable load
- Serverless for sporadic workloads

### Storage Optimization
- Lifecycle policies (move cold data to cheaper tiers)
- Compression
- Deduplication
- Delete unused snapshots
- Right-size provisioned IOPS

### Network Optimization
- Minimize cross-region traffic
- Use CDN for static assets
- Compress API responses
- Batch requests where possible

## 2. Cost Allocation

Tag everything:
```
Environment: production/staging/development
Team: platform/backend/ml
Service: user-service/order-service
CostCenter: engineering/marketing
```

## 3. FinOps Practices

- Regular cost reviews (weekly/monthly)
- Anomaly detection and alerts
- Showback/chargeback to teams
- Committed use discounts
- Spot instance strategies

---

# PART XXII: COMPLIANCE & GOVERNANCE

## 1. Data Classification

| Level | Description | Examples | Controls |
|-------|-------------|----------|----------|
| Public | Freely available | Marketing content | None |
| Internal | Company-only | Internal docs | Authentication |
| Confidential | Restricted access | Customer data | Encryption, RBAC |
| Restricted | Highly sensitive | PII, financial | Encryption, audit, MFA |

## 2. Regulatory Requirements

| Regulation | Scope | Key Requirements |
|------------|-------|------------------|
| GDPR | EU residents | Consent, right to erasure, data portability |
| CCPA | CA residents | Disclosure, opt-out, non-discrimination |
| HIPAA | Healthcare (US) | PHI protection, audit trails |
| PCI-DSS | Payment cards | Encryption, network segmentation |
| SOC 2 | Service providers | Security, availability, confidentiality |

## 3. Audit Logging Requirements

Log every access to sensitive data:
```json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "actor": "user:alice@example.com",
  "action": "read",
  "resource": "customer:123",
  "resource_type": "PII",
  "ip_address": "192.168.1.1",
  "user_agent": "Mozilla/5.0...",
  "result": "success"
}
```

**Retention:** Typically 1-7 years depending on regulation.
**Protection:** Logs must be immutable and tamper-evident.

---

# PART XXIII: DISASTER RECOVERY

## 1. Recovery Objectives

| Metric | Definition | Typical Values |
|--------|------------|----------------|
| **RTO** | Recovery Time Objective | Minutes to hours |
| **RPO** | Recovery Point Objective | Seconds to hours |

### RTO/RPO Trade-offs
```
Lower RTO/RPO = Higher Cost

Hot standby: RTO/RPO ~ 0, Cost: 2x
Warm standby: RTO ~ minutes, RPO ~ minutes, Cost: 1.5x
Cold standby: RTO ~ hours, RPO ~ hours, Cost: 1.1x
Backup only: RTO ~ days, RPO ~ hours, Cost: 1.05x
```

## 2. Backup Strategies

### 3-2-1 Rule
- **3** copies of data
- **2** different storage types
- **1** offsite location

### Backup Types
| Type | Size | Restore Time |
|------|------|--------------|
| Full | Large | Fast |
| Incremental | Small | Slow (chain) |
| Differential | Medium | Medium |

## 3. DR Testing

| Test Type | Scope | Frequency |
|-----------|-------|-----------|
| Tabletop | Discussion only | Quarterly |
| Walkthrough | Partial execution | Bi-annually |
| Simulation | Full failover | Annually |
| Full test | Actual failover | Every 2 years |

---

# PART XXIV: DOCUMENTATION STANDARDS

## 1. Documentation Types

| Type | Audience | Update Frequency |
|------|----------|------------------|
| Architecture diagrams | Everyone | On change |
| API reference | Developers | Every release |
| Runbooks | Operators | On incident learnings |
| ADRs | Future engineers | On decision |
| Onboarding guides | New team members | Quarterly |

## 2. Architecture Diagram Standards

### C4 Model Levels
```
Level 1: System Context  - How system fits in the world
Level 2: Container       - High-level technology choices
Level 3: Component       - Internal structure of containers
Level 4: Code            - Implementation details (optional)
```

### Diagram Requirements
- Clear title and date
- Legend for all symbols
- Clear boundaries
- Named connections with protocols
- Version controlled

## 3. Runbook Structure

```markdown
# Service Name: Runbook

## Overview
Brief description of the service.

## Dependencies
- Database: PostgreSQL (prod-db-1)
- Queue: RabbitMQ (prod-mq-1)
- External: Stripe API

## Common Issues

### Issue: High latency
**Symptoms:** P99 latency > 500ms
**Diagnosis:**
1. Check database slow query log
2. Check connection pool saturation
**Resolution:**
1. Scale read replicas
2. Clear cache if stale

### Issue: Out of memory
**Symptoms:** OOM killed, pod restarts
**Diagnosis:** Check memory metrics
**Resolution:** Increase limits or fix memory leak
```

---

# PART XXV: NETWORKING DEEP DIVE

## 1. The OSI Model in Practice

Understanding networking layers helps debug distributed systems:

| Layer | Protocol | Debugging Tools |
|-------|----------|-----------------|
| Application | HTTP, gRPC | curl, Postman |
| Transport | TCP, UDP | netstat, tcpdump |
| Network | IP | ping, traceroute |
| Data Link | Ethernet | arp, switch logs |

## 2. TCP Fundamentals

### Connection Lifecycle
TCP uses a three-way handshake for connection establishment and a four-way handshake for termination. Understanding this helps debug connection issues.

### Congestion Control
TCP implements congestion control through:
- Slow start: Exponential window growth
- Congestion avoidance: Linear growth after threshold
- Fast retransmit: Retransmit on duplicate ACKs

### Tuning Parameters
| Parameter | Default | High-Traffic Tuning |
|-----------|---------|---------------------|
| tcp_keepalive_time | 7200s | 60s |
| tcp_fin_timeout | 60s | 15s |
| somaxconn | 128 | 4096 |

## 3. DNS Resolution

DNS resolution follows a hierarchical process:
```
Client → Recursive Resolver → Root NS → TLD NS → Authoritative NS
```

### Record Types
- A/AAAA: IP addresses
- CNAME: Aliases
- MX: Mail servers
- TXT: Arbitrary text (SPF, DKIM)
- SRV: Service discovery

### Caching Layers
Caching occurs at multiple levels: browser, OS, resolver, and authoritative server. Each layer respects TTL values.

## 4. Load Balancing

### Algorithms
| Algorithm | Pros | Cons |
|-----------|------|------|
| Round Robin | Simple | Ignores load |
| Least Connections | Load-aware | State required |
| IP Hash | Sticky sessions | Uneven distribution |
| Consistent Hashing | Minimal redistribution | Complexity |

### Health Checking
Load balancers should implement:
- Active health checks (periodic probes)
- Passive health checks (monitor real traffic)
- Graceful degradation (remove unhealthy nodes)

---

# PART XXVI: CONTAINER ORCHESTRATION

## 1. Kubernetes Architecture

Kubernetes consists of a control plane and worker nodes:

**Control Plane:**
- API Server: Entry point for all operations
- Scheduler: Places pods on nodes
- Controller Manager: Reconciliation loops
- etcd: Distributed key-value store

**Worker Nodes:**
- kubelet: Node agent
- kube-proxy: Network proxy
- Container runtime: Docker, containerd

## 2. Core Objects

### Pod
The smallest deployable unit. Contains one or more containers sharing network and storage.

### Deployment
Manages pod replicas and rolling updates. Supports rollback.

### Service
Provides stable network endpoint. Types: ClusterIP, NodePort, LoadBalancer.

### ConfigMap/Secret
External configuration. Secrets are base64 encoded (not encrypted by default).

## 3. Resource Management

### Requests vs Limits
- **Requests:** Guaranteed minimum, used for scheduling
- **Limits:** Maximum allowed, enforced at runtime

### Quality of Service
- Guaranteed: requests == limits
- Burstable: requests < limits
- BestEffort: no requests/limits

## 4. Networking

### Network Policies
Define ingress/egress rules for pods. Default: all traffic allowed.

### Service Mesh
Sidecar proxies handle:
- Mutual TLS
- Traffic management
- Observability
- Resilience patterns

---

# PART XXVII: OBSERVABILITY

## 1. The Three Pillars

### Metrics
Aggregated numerical data over time.
- Counters: monotonically increasing
- Gauges: point-in-time values
- Histograms: distribution of values

### Logs
Discrete events with context.
- Structured (JSON): machine-parseable
- Unstructured: human-readable

### Traces
Request flow across services.
- Trace: end-to-end request
- Span: single operation within trace

## 2. Alerting Strategy

### Alert Quality
- High precision: low false positives
- High recall: catch real incidents
- Actionable: clear remediation path

### SLO-Based Alerting
Alert on error budget burn rate rather than raw metrics:
- Fast burn (1h window): page immediately
- Slow burn (6h window): create ticket

## 3. Incident Response

### Process
Detection → Triage → Mitigation → Resolution → Postmortem

### Postmortem Culture
- Blameless
- Focus on systems, not individuals
- Action items with owners and deadlines
- Shared learnings

---

# PART XXVIII: SECURITY

## 1. Threat Modeling

### STRIDE Framework
| Threat | Mitigation |
|--------|------------|
| Spoofing | Authentication |
| Tampering | Integrity checks |
| Repudiation | Audit logging |
| Information Disclosure | Encryption |
| Denial of Service | Rate limiting |
| Elevation of Privilege | Authorization |

## 2. Authentication

### Token-Based (JWT)
Structure: Header.Payload.Signature

Best practices:
- Short expiration (15 min - 1 hour)
- Use asymmetric keys for distributed systems
- Validate all claims

### OAuth 2.0
Use Authorization Code + PKCE for web/mobile apps.
Use Client Credentials for machine-to-machine.

## 3. Cryptography

### Recommended Algorithms (2025)
| Purpose | Algorithm |
|---------|-----------|
| Symmetric encryption | AES-256-GCM |
| Password hashing | Argon2id |
| Digital signatures | Ed25519 |
| Key exchange | X25519 |

### What to Avoid
- MD5, SHA-1 (broken)
- DES, 3DES (weak)
- Custom cryptography

## 4. Application Security

### Common Vulnerabilities
- SQL Injection: Use parameterized queries
- XSS: Escape output
- CSRF: Use tokens
- SSRF: Validate URLs

### Security Headers
```
Strict-Transport-Security: max-age=31536000
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
```

---

# PART XXIX: PERFORMANCE

## 1. Testing Types

| Type | Goal | Duration |
|------|------|----------|
| Load | Verify SLAs | Minutes |
| Stress | Find limits | Until failure |
| Soak | Find leaks | Hours/days |
| Spike | Handle bursts | Minutes |

## 2. Profiling

### CPU Profiling
Identify hot paths and optimization opportunities.

### Memory Profiling
Find allocations and potential memory leaks.

### Flame Graphs
Visualize call stacks with time spent at each level.

## 3. Database Optimization

### Query Optimization
- Check execution plans
- Add appropriate indexes
- Avoid N+1 queries

### Connection Pooling
Reuse connections to reduce overhead.
Formula: connections = (cores * 2) + spindle_count

## 4. Caching

### Strategies
| Strategy | Description |
|----------|-------------|
| Cache-aside | Application manages cache |
| Write-through | Cache updated on write |
| Write-behind | Async cache-to-DB sync |

### Invalidation
- TTL-based: simple but eventually consistent
- Event-driven: complex but accurate

---

# PART XXX: SYSTEM DESIGN PATTERNS

## 1. Estimation

### Traffic
```
100M users × 10% DAU × 10 actions = 100M/day
100M / 86400 ≈ 1,200 req/sec
Peak: 3x average ≈ 3,600 req/sec
```

### Storage
```
100M users × 1KB = 100GB profiles
100M actions/day × 500B × 365 = 18TB/year
```

## 2. Common Problems

### URL Shortener
- Hash URL to short code
- Store mapping
- Handle collisions
- Analytics optional

### Rate Limiter
- Token bucket for smooth limiting
- Sliding window for accuracy
- Redis for distributed state

### Notification System
- Queue for reliability
- Workers for each channel
- Retry with backoff

## 3. Interview Framework

1. **Clarify** (5 min): Requirements, constraints
2. **Estimate** (5 min): Scale, traffic, storage
3. **Design** (10 min): High-level components
4. **Deep Dive** (15 min): 2-3 components in detail
5. **Wrap Up** (5 min): Summarize, operational concerns

---

# CLOSING NOTES

This framework represents accumulated engineering wisdom across many domains. It is:

- **Descriptive:** How systems actually work
- **Prescriptive:** How systems should be built
- **Pragmatic:** Trade-offs explicitly acknowledged

No system perfectly implements all patterns. Engineering is the art of choosing which principles to apply given constraints.

The principles are stable. The technologies change. Focus on understanding the principles.

---

*This Framework is Open Source.*
*Designed for building systems that work.*

---

# APPENDIX A: DEEP DIVE INTO DISTRIBUTED SYSTEMS

## 1. The Fallacies of Distributed Computing

Peter Deutsch identified eight fallacies that engineers often assume (incorrectly):

1. **The network is reliable.** Networks fail. Packets are lost, duplicated, or corrupted. Design for failure.

2. **Latency is zero.** Every network call adds latency. 1ms in datacenter, 150ms cross-continent. Minimize round trips.

3. **Bandwidth is infinite.** Bandwidth costs money and has limits. Compress data, batch requests.

4. **The network is secure.** Assume all network traffic can be intercepted. Use TLS everywhere.

5. **Topology doesn't change.** Network topology changes due to scaling, failures, and migrations. Use service discovery.

6. **There is one administrator.** Multiple teams manage different parts. Document interfaces clearly.

7. **Transport cost is zero.** Cloud providers charge for data transfer. Design to minimize cross-zone traffic.

8. **The network is homogeneous.** Different parts use different protocols and technologies. Build adapters.

## 2. Consensus Algorithms

### Paxos
The foundational consensus algorithm. Guarantees safety but complex to understand and implement.

**Roles:**
- Proposers: Suggest values
- Acceptors: Vote on values
- Learners: Learn decided values

**Phases:**
1. Prepare: Proposer asks acceptors to promise
2. Promise: Acceptors promise if proposal number is highest seen
3. Accept: Proposer sends value with highest proposal number
4. Accepted: Value is chosen when majority accepts

### Raft
Designed to be more understandable than Paxos. Used in etcd, Consul.

**Components:**
- Leader Election: One leader at a time
- Log Replication: Leader replicates log to followers
- Safety: Committed entries are never lost

**States:**
- Leader: Handles all client requests
- Follower: Responds to leader
- Candidate: Seeks to become leader

**Election Process:**
1. Follower times out (no heartbeat from leader)
2. Becomes candidate, increments term
3. Votes for self, requests votes from others
4. Wins if receives majority of votes

### Practical Applications

| System | Algorithm | Use Case |
|--------|-----------|----------|
| etcd | Raft | Kubernetes state |
| ZooKeeper | ZAB (Paxos-like) | Coordination |
| CockroachDB | Raft | Distributed SQL |
| Consul | Raft | Service discovery |

## 3. Time in Distributed Systems

### The Problem with Clocks
Physical clocks drift. Network Time Protocol (NTP) can only synchronize to within milliseconds. This is problematic when ordering events.

### Logical Clocks

**Lamport Clocks:**
- Counter incremented on each event
- On message send: attach counter
- On message receive: max(local, received) + 1
- Partial ordering only

**Vector Clocks:**
- Each node maintains vector of counters
- Can determine if events are causally related
- Storage grows with number of nodes

### Hybrid Logical Clocks (HLC)
Combines physical time with logical counters. Used in CockroachDB and others.

### TrueTime (Google Spanner)
Uses GPS and atomic clocks to bound clock uncertainty. Allows global consistent reads.

## 4. Distributed Transactions

### Two-Phase Commit (2PC)

**Phase 1: Prepare**
1. Coordinator asks all participants: "Can you commit?"
2. Participants prepare and respond: "Yes" or "No"

**Phase 2: Commit/Abort**
1. If all "Yes": Coordinator sends "Commit"
2. If any "No" or timeout: Coordinator sends "Abort"
3. Participants acknowledge

**Problems:**
- Blocking: Participants block if coordinator fails
- Coordinator is single point of failure
- Performance: Two round trips minimum

### Three-Phase Commit (3PC)
Adds "pre-commit" phase to avoid blocking. Still has edge cases with network partitions.

### Saga Pattern (Preferred)
Instead of distributed transactions, use compensating transactions:

```
Order Saga:
1. Create Order (pending)
2. Reserve Inventory
3. Process Payment
4. Confirm Order

On failure at step 3:
- Compensate step 2: Release Inventory
- Compensate step 1: Cancel Order
```

## 5. Consistency Models Deep Dive

### Linearizability (Strongest)
Operations appear instantaneous. Every read sees the most recent write.

**Properties:**
- Total order of operations
- Real-time ordering respected
- Expensive to implement

### Sequential Consistency
Operations appear in some total order that respects each process's program order.

**Difference from linearizability:** Real-time ordering not required.

### Causal Consistency
Operations that are causally related are seen in the same order by all processes.

**Example:** If A causes B, everyone sees A before B. Unrelated operations may be seen in different orders.

### Eventual Consistency
All replicas eventually converge to the same value. No guarantees on timing.

**Conflict Resolution:**
- Last-Write-Wins (LWW): Timestamp-based, data loss possible
- Multi-Value: Return all conflicting values, application resolves
- CRDTs: Mathematically guaranteed convergence

## 6. CRDTs (Conflict-Free Replicated Data Types)

### G-Counter (Grow-Only Counter)
```
Node A: {A: 5, B: 3, C: 2}
Node B: {A: 5, B: 4, C: 2}

Merge: {A: max(5,5), B: max(3,4), C: max(2,2)}
     = {A: 5, B: 4, C: 2}

Total = 5 + 4 + 2 = 11
```

### PN-Counter (Positive-Negative Counter)
Two G-Counters: one for increments, one for decrements.

### LWW-Register (Last-Write-Wins Register)
Store value with timestamp. On conflict, higher timestamp wins.

### OR-Set (Observed-Remove Set)
Add and remove operations with unique tags. Can add and remove same element.

---

# APPENDIX B: DATABASE INTERNALS

## 1. Storage Engines

### B-Tree
The most common storage structure for databases.

**Structure:**
- Balanced tree with high fanout
- Keys sorted at each level
- Leaf nodes contain data or pointers

**Properties:**
- O(log n) reads and writes
- Good for range queries
- In-place updates

### LSM-Tree (Log-Structured Merge Tree)
Used in RocksDB, Cassandra, LevelDB.

**Structure:**
- Writes go to in-memory buffer (memtable)
- When full, flush to disk as immutable SSTable
- Background compaction merges SSTables

**Properties:**
- O(1) writes (append-only)
- Reads may check multiple levels
- Good for write-heavy workloads

### Comparison

| Property | B-Tree | LSM-Tree |
|----------|--------|----------|
| Write pattern | Random I/O | Sequential I/O |
| Read pattern | Single lookup | Multiple lookups |
| Space amplification | Low | Higher (before compaction) |
| Write amplification | Higher | Lower |
| Best for | Read-heavy | Write-heavy |

## 2. Indexing Structures

### B+ Tree Index
Standard database index. Keys in internal nodes, data in leaf nodes.

### Hash Index
O(1) lookups. Cannot do range queries. Used for equality predicates.

### Bitmap Index
For low-cardinality columns. Each distinct value has a bitmap.

### Bloom Filter
Probabilistic structure. Can say "definitely not present" or "maybe present". Used to avoid unnecessary disk reads.

### Inverted Index
Maps terms to documents. Used for full-text search.

## 3. Query Execution

### Query Planning

```sql
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'US'
AND o.total > 100;
```

**Possible Plans:**
1. Scan orders, filter, then join customers
2. Scan customers, filter country, then join orders
3. Use index on customers.country, then join

**Cost Estimation:**
- Estimated rows at each step
- I/O cost (disk reads)
- CPU cost (comparisons, hashing)

### Join Algorithms

**Nested Loop Join:**
```
for each row in outer:
    for each row in inner:
        if match: emit
```
O(n*m). Good when inner is small or indexed.

**Hash Join:**
```
build hash table from smaller relation
probe with larger relation
```
O(n+m). Requires memory for hash table.

**Sort-Merge Join:**
```
sort both relations
merge like merge sort
```
O(n log n + m log m). Good when already sorted.

## 4. Transaction Isolation

### Read Phenomena

| Phenomenon | Description |
|------------|-------------|
| Dirty Read | Read uncommitted data |
| Non-Repeatable Read | Same query, different results |
| Phantom Read | New rows appear in range query |
| Write Skew | Concurrent writes cause invalid state |

### Isolation Levels

| Level | Dirty Read | Non-Repeatable | Phantom | Write Skew |
|-------|------------|----------------|---------|------------|
| Read Uncommitted | Possible | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible | Possible |
| Serializable | Prevented | Prevented | Prevented | Prevented |

### Implementation Approaches

**Locking (2PL):**
- Shared locks for reads
- Exclusive locks for writes
- Two-phase: acquire all locks, then release all

**MVCC (Multi-Version Concurrency Control):**
- Keep multiple versions of data
- Readers see snapshot
- Writers create new version
- No read-write blocking

---

# APPENDIX C: ARCHITECTURE PATTERNS

## 1. Microservices vs Monolith

### When to Use Monolith
- Small team (< 10 engineers)
- Well-understood domain
- Tight deadlines
- Simple deployment requirements

### When to Use Microservices
- Large organization (> 50 engineers)
- Independent team scaling
- Different technology requirements per service
- Independent deployment needed

### Migration Path
1. Start with modular monolith
2. Identify service boundaries
3. Extract services incrementally
4. Use Strangler Fig pattern

## 2. Event-Driven Architecture

### Event Types

**Domain Events:**
- Business-meaningful events
- "OrderPlaced", "PaymentReceived"
- Published by domain logic

**Integration Events:**
- Cross-service communication
- May aggregate domain events
- Published at service boundary

### Event Schemas

**Envelope:**
```json
{
  "eventId": "uuid",
  "eventType": "OrderPlaced",
  "timestamp": "2025-01-15T10:30:00Z",
  "source": "order-service",
  "data": { ... }
}
```

**Versioning:**
- Add fields as optional (backward compatible)
- Create new event type for breaking changes
- Include schema version in envelope

### Delivery Guarantees

| Guarantee | Description | Implementation |
|-----------|-------------|----------------|
| At-most-once | May lose messages | Fire and forget |
| At-least-once | May duplicate | Retry + idempotency |
| Exactly-once | Neither loss nor duplication | Transactional outbox |

### Transactional Outbox Pattern
```
Transaction:
1. Update business data
2. Write event to outbox table
3. Commit

Background:
1. Poll outbox table
2. Publish to message queue
3. Mark as published
```

## 3. API Gateway Pattern

### Responsibilities
- Request routing
- Authentication/Authorization
- Rate limiting
- Request/Response transformation
- Caching
- Monitoring

### Implementation Options

| Option | Pros | Cons |
|--------|------|------|
| Kong | Feature-rich, plugins | Complexity |
| Envoy | High performance | Configuration |
| AWS API Gateway | Managed | Vendor lock-in |
| Custom | Full control | Build/maintain cost |

## 4. Service Mesh

### Data Plane (Sidecar Proxies)
- Handle all network traffic
- Mutual TLS
- Load balancing
- Circuit breaking
- Observability

### Control Plane
- Configuration distribution
- Certificate management
- Service discovery
- Policy enforcement

### Comparison

| Feature | Istio | Linkerd | Consul Connect |
|---------|-------|---------|----------------|
| Complexity | High | Low | Medium |
| Performance | Good | Better | Good |
| Features | Most | Core | Good |
| Learning Curve | Steep | Gentle | Medium |

---

# APPENDIX D: OPERATIONAL EXCELLENCE

## 1. SRE Principles

### Service Level Objectives (SLOs)
Define what "reliable enough" means:
- Availability: 99.9% uptime (8.77 hours/year downtime)
- Latency: 95th percentile < 200ms
- Error rate: < 0.1% of requests

### Error Budgets
If SLO is 99.9%, error budget is 0.1%.
- Budget available: ship features, take risks
- Budget exhausted: focus on reliability

### Toil
Manual, repetitive, automatable work. Target: < 50% of SRE time.

Reduce toil through:
- Automation
- Self-healing systems
- Better tooling

## 2. Incident Management

### Severity Levels

| Level | Impact | Response |
|-------|--------|----------|
| SEV1 | Customer-facing outage | Page on-call, war room |
| SEV2 | Major degradation | Page on-call |
| SEV3 | Minor impact | Business hours |
| SEV4 | No customer impact | Best effort |

### Incident Commander Role
- Coordinates response
- Delegates tasks
- Communicates status
- Does NOT debug

### Communication Template
```
[INCIDENT] Service X degraded

Impact: 10% of requests failing
Start time: 10:30 UTC
Current status: Investigating

Next update: 11:00 UTC
```

## 3. Postmortem Process

### Timeline
- 24-48 hours: Draft postmortem
- 1 week: Review with team
- 2 weeks: Action items assigned
- Ongoing: Track completion

### Template Structure
1. Summary
2. Impact
3. Timeline
4. Root Cause
5. What Went Well
6. What Went Poorly
7. Action Items

### Blameless Culture
- Focus on systems, not people
- "What allowed this to happen?" not "Who caused this?"
- Safe to report mistakes
- Share learnings widely

## 4. Capacity Planning

### Signals
- CPU utilization > 70% sustained
- Memory utilization > 80%
- Disk utilization > 85%
- Queue depth growing

### Forecasting
```
Current: 1000 req/sec
Growth: 10% month-over-month
6 months: 1000 * (1.1)^6 = 1771 req/sec
```

### Provisioning Strategy
- Lead time for new capacity
- Buffer for unexpected spikes
- Cost optimization (reserved vs on-demand)

---

# APPENDIX E: TECHNOLOGY SELECTION

## 1. Decision Framework

### Build vs Buy
| Factor | Build | Buy |
|--------|-------|-----|
| Core competency | Yes | No |
| Differentiation | High | Low |
| Team expertise | High | Low |
| Time to market | Low priority | High priority |
| Long-term cost | Lower | Higher |

### Technology Evaluation

**Criteria:**
1. Does it solve the problem?
2. Can we operate it?
3. Is it mature enough?
4. Is there community/support?
5. What's the lock-in risk?

### Proof of Concept
- Time-boxed (1-2 weeks)
- Clear success criteria
- Test realistic scenarios
- Evaluate operational aspects

## 2. Language Selection

| Language | Strengths | Best For |
|----------|-----------|----------|
| Go | Concurrency, simplicity | Infrastructure, CLIs |
| Rust | Performance, safety | Systems, performance-critical |
| Python | Productivity, ecosystem | Scripts, ML, prototypes |
| TypeScript | Type safety, ecosystem | Frontend, Node.js backend |
| Java/Kotlin | Enterprise, mature | Large applications |

## 3. Database Selection

| Database | Type | Best For |
|----------|------|----------|
| PostgreSQL | Relational | General purpose |
| MySQL | Relational | Web applications |
| MongoDB | Document | Flexible schema |
| Redis | Key-Value | Caching, sessions |
| Cassandra | Wide-column | Time-series, high write |
| Neo4j | Graph | Relationships |
| Elasticsearch | Search | Full-text search |

## 4. Message Queue Selection

| Queue | Strengths | Best For |
|-------|-----------|----------|
| Kafka | Throughput, durability | Event streaming |
| RabbitMQ | Flexibility, routing | Task queues |
| SQS | Managed, simple | AWS workloads |
| Redis Streams | Simple, fast | Lightweight streaming |

---

# CLOSING NOTES

This framework represents accumulated engineering wisdom across many domains. It is:

- **Descriptive:** How systems actually work
- **Prescriptive:** How systems should be built
- **Pragmatic:** Trade-offs explicitly acknowledged

No system perfectly implements all patterns. Engineering is the art of choosing which principles to apply given constraints.

The principles are stable. The technologies change. Focus on understanding the principles.

---

*This Framework is Open Source.*
*Designed for building systems that work.*

---

# APPENDIX F: CLOUD ARCHITECTURE

## 1. Multi-Cloud Strategy

### Why Multi-Cloud
- Avoid vendor lock-in
- Best-of-breed services
- Geographic requirements
- Negotiating leverage

### Challenges
- Complexity multiplied
- Different APIs and concepts
- Networking between clouds
- Skill requirements

### Pragmatic Approach
- Primary cloud for most workloads
- Secondary for specific requirements
- Abstract where practical
- Accept some lock-in for productivity

## 2. Cloud Native Principles

### The Twelve-Factor App

1. **Codebase:** One codebase, many deploys
2. **Dependencies:** Explicitly declare and isolate
3. **Config:** Store config in environment
4. **Backing Services:** Treat as attached resources
5. **Build, Release, Run:** Strictly separate stages
6. **Processes:** Execute as stateless processes
7. **Port Binding:** Export services via port
8. **Concurrency:** Scale out via process model
9. **Disposability:** Fast startup and graceful shutdown
10. **Dev/Prod Parity:** Keep environments similar
11. **Logs:** Treat as event streams
12. **Admin Processes:** Run as one-off processes

### Beyond Twelve Factors

- **API First:** Design APIs before implementation
- **Telemetry:** Built-in observability
- **Security:** Security as code
- **Feature Flags:** Decouple deploy from release

## 3. Serverless Architecture

### When to Use
- Sporadic traffic patterns
- Event-driven workloads
- Rapid prototyping
- Cost optimization for low traffic

### When to Avoid
- Predictable high traffic
- Long-running processes
- Complex state management
- Strict latency requirements (cold starts)

### Patterns

**Function Composition:**
```
API Gateway → Lambda → DynamoDB
                    → S3
                    → SQS → Lambda
```

**Event Processing:**
```
Kinesis/Kafka → Lambda → Elasticsearch
                      → S3 (archive)
```

### Cold Start Mitigation
- Provisioned concurrency
- Keep functions warm (scheduled pings)
- Optimize package size
- Use compiled languages (Go, Rust)

## 4. Infrastructure as Code

### Tools Comparison

| Tool | Language | State | Cloud Support |
|------|----------|-------|---------------|
| Terraform | HCL | Remote state | All major |
| Pulumi | TypeScript, Python, Go | Remote state | All major |
| CloudFormation | YAML/JSON | AWS managed | AWS only |
| CDK | TypeScript, Python | CloudFormation | AWS (primary) |
| Crossplane | YAML (K8s) | Kubernetes | All major |

### Best Practices

**Module Structure:**
```
infrastructure/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
└── shared/
```

**State Management:**
- Remote state (S3, GCS, Azure Blob)
- State locking (DynamoDB, GCS)
- Separate state per environment
- Never commit state files

**Drift Detection:**
- Regular plan comparisons
- Automated drift alerts
- Reconciliation processes

---

# APPENDIX G: DATA ENGINEERING

## 1. Data Pipeline Architecture

### Batch Processing
```
Sources → Extract → Transform → Load → Warehouse
         (daily)   (Spark)    (batch) (BigQuery)
```

**Tools:** Apache Spark, dbt, Airflow

### Stream Processing
```
Sources → Ingest → Process → Store → Serve
         (Kafka)  (Flink)   (real-time)
```

**Tools:** Apache Kafka, Apache Flink, Spark Streaming

### Lambda Architecture
Combine batch and stream for complete picture:
```
              ┌──────────────────────────────────┐
              │            Serving Layer          │
              └──────────────┬───────────────────┘
                             │
         ┌───────────────────┴───────────────────┐
         │                                       │
    ┌────┴────┐                           ┌──────┴─────┐
    │ Batch   │                           │   Speed    │
    │ Layer   │                           │   Layer    │
    └────┬────┘                           └──────┬─────┘
         │                                       │
         └───────────────┬───────────────────────┘
                         │
                  ┌──────┴──────┐
                  │ Raw Data    │
                  └─────────────┘
```

### Kappa Architecture
Simplified: stream processing only, replay for reprocessing.

## 2. Data Warehouse Design

### Dimensional Modeling

**Fact Tables:**
- Measurable events
- Foreign keys to dimensions
- Additive metrics

**Dimension Tables:**
- Descriptive attributes
- Slowly changing dimensions (SCD)
- Denormalized for query performance

### Star Schema
```
            ┌────────────┐
            │ dim_date   │
            └──────┬─────┘
                   │
┌────────────┐     │     ┌────────────┐
│dim_product │─────┼─────│dim_customer│
└────────────┘     │     └────────────┘
                   │
            ┌──────┴─────┐
            │ fact_sales │
            └────────────┘
```

### Slowly Changing Dimensions

| Type | Description | Use Case |
|------|-------------|----------|
| Type 1 | Overwrite | No history needed |
| Type 2 | Add row with dates | Full history |
| Type 3 | Add column | Limited history |

## 3. Data Quality

### Dimensions of Quality

| Dimension | Description | Example Check |
|-----------|-------------|---------------|
| Completeness | No missing values | NULL count < threshold |
| Accuracy | Correct values | Value in valid range |
| Consistency | Same across sources | Cross-source comparison |
| Timeliness | Data is current | Max age < threshold |
| Uniqueness | No duplicates | Unique key check |

### Testing Framework
```python
def test_no_null_customer_id():
    assert df.filter(col("customer_id").isNull()).count() == 0

def test_valid_order_total():
    assert df.filter(col("total") < 0).count() == 0

def test_referential_integrity():
    orphans = orders.join(customers, "customer_id", "left_anti")
    assert orphans.count() == 0
```

### Data Observability
- Row counts over time
- Schema changes
- Value distribution changes
- Freshness monitoring

## 4. Data Governance

### Metadata Management
- Data catalog (discovery)
- Data lineage (tracing)
- Data dictionary (definitions)

### Access Control
- Role-based access
- Column-level security
- Row-level security
- Dynamic data masking

### Compliance
- PII identification and handling
- Data retention policies
- Right to erasure implementation
- Audit logging

---

# APPENDIX H: MACHINE LEARNING OPERATIONS

## 1. ML Lifecycle

```
Data → Feature Engineering → Training → Evaluation → Deployment → Monitoring
 │           │                  │           │             │           │
 └───────────┴──────────────────┴───────────┴─────────────┴───────────┘
                              Iterate
```

## 2. Feature Stores

### Architecture
```
┌─────────────────────────────────────────────┐
│              Feature Store                   │
├─────────────────────────────────────────────┤
│  Feature Definitions (code)                 │
├─────────────────────────────────────────────┤
│  Offline Store    │    Online Store         │
│  (training)       │    (serving)            │
│  - Warehouse      │    - Redis/DynamoDB     │
└─────────────────────────────────────────────┘
```

### Benefits
- Feature reuse across models
- Training-serving consistency
- Point-in-time correctness
- Feature documentation

## 3. Model Registry

### Capabilities
- Version control for models
- Metadata (metrics, parameters)
- Stage management (staging, production)
- Lineage (data, code, environment)

### MLflow Example
```python
with mlflow.start_run():
    mlflow.log_param("learning_rate", 0.01)
    mlflow.log_metric("accuracy", 0.95)
    mlflow.sklearn.log_model(model, "model")
```

## 4. Model Serving

### Patterns

**Batch Prediction:**
```
Schedule → Load Model → Score Data → Write Results
```

**Online Prediction:**
```
Request → Load Model → Inference → Response
           (cached)    (<100ms)
```

**Embedded:**
```
Model compiled into application (edge devices)
```

### Serving Frameworks

| Framework | Strengths | Best For |
|-----------|-----------|----------|
| TensorFlow Serving | TF optimized | TensorFlow models |
| TorchServe | PyTorch optimized | PyTorch models |
| Triton | Multi-framework, GPU | High throughput |
| BentoML | Simple, flexible | General purpose |
| Seldon | Kubernetes native | K8s deployments |

## 5. Model Monitoring

### What to Monitor

**Data Quality:**
- Input distribution shift
- Missing values
- Out-of-range values

**Model Performance:**
- Prediction distribution
- Accuracy metrics (if labels available)
- Latency

**Business Metrics:**
- Click-through rate
- Conversion rate
- Revenue impact

### Drift Detection
```
Training Distribution vs Production Distribution

Statistical Tests:
- Kolmogorov-Smirnov test
- Population Stability Index (PSI)
- Jensen-Shannon divergence
```

---

# APPENDIX I: DEVELOPER EXPERIENCE

## 1. Development Environment

### Standardization
- Dockerized development environments
- Infrastructure as code for dev
- Pre-configured IDE settings
- Shared dotfiles

### Local Development
```
┌─────────────────────────────────────┐
│         Developer Machine            │
│  ┌─────────────────────────────┐    │
│  │      Docker Compose         │    │
│  │  ┌─────┐ ┌─────┐ ┌──────┐  │    │
│  │  │ App │ │ DB  │ │Cache │  │    │
│  │  └─────┘ └─────┘ └──────┘  │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

### Remote Development
- GitHub Codespaces
- GitPod
- VS Code Remote
- Cloud-based IDEs

## 2. CI/CD Pipeline

### Pipeline Stages
```
Commit → Build → Test → Security → Deploy → Verify
           │       │        │         │        │
           │       │        │         │        └─ Smoke tests
           │       │        │         └─ Canary/Blue-Green
           │       │        └─ SAST, dependency scan
           │       └─ Unit, integration, contract
           └─ Compile, Docker build
```

### Pipeline as Code
```yaml
# .github/workflows/ci.yml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: make build
      - name: Test
        run: make test
```

### Best Practices
- Fast feedback (< 10 min for commit)
- Parallelization
- Caching (dependencies, Docker layers)
- Fail fast (lint before test before deploy)

## 3. Code Review

### What to Review
- Correctness
- Design
- Readability
- Test coverage
- Security implications
- Performance

### What NOT to Review (Automate Instead)
- Formatting (use formatters)
- Linting rules (use linters)
- Import ordering (use tools)
- Common patterns (use templates)

### Review Etiquette
- Be kind and constructive
- Explain the "why"
- Differentiate blocking vs non-blocking
- Respond promptly

## 4. Documentation

### Types
| Type | Audience | Update Trigger |
|------|----------|----------------|
| README | New developers | Feature changes |
| API docs | Consumers | API changes |
| Architecture | Team | Design changes |
| Runbooks | Operators | Incident learnings |
| ADRs | Future developers | Decisions |

### Documentation as Code
- Store with code (version control)
- Review with code (PRs)
- Test with code (links, examples)
- Deploy with code (CI/CD)

---

# FINAL CLOSING NOTES

This comprehensive framework covers the breadth and depth of modern software engineering. From fundamental distributed systems concepts to practical operational concerns, it provides a reference for building reliable, scalable systems.

**Key Takeaways:**

1. **Principles over tools:** Technologies change, principles endure.

2. **Trade-offs everywhere:** Every decision has costs and benefits. Make them explicitly.

3. **Simplicity wins:** The best architecture is the simplest one that meets requirements.

4. **Iterate:** Start simple, evolve as needs become clear.

5. **Measure:** What gets measured gets improved.

6. **Automate:** Reduce toil, increase reliability.

7. **Learn:** Incidents are learning opportunities. Share knowledge widely.

Remember: The goal is not to implement every pattern in this framework. The goal is to understand the patterns well enough to choose the right ones for your specific situation.

Build systems that work. Keep them working. Make them better.

---

*This Framework is Open Source.*
*Designed for building systems that work.*
