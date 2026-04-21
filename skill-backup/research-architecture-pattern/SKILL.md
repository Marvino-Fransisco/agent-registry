---
name: research-architecture-pattern
description: >
  Use this skill to research, evaluate, and recommend software architecture patterns for
  a given problem. Trigger when the user asks about system design choices like "should we
  use microservices or monolith?", "what pattern should we use for X?", "how do companies
  like X handle Y at scale?", "event-driven vs request-response for this use case?",
  "CQRS vs traditional CRUD?", or any question about structuring systems, services, or
  data flows. Also trigger when the user is designing a new feature and needs to understand
  proven patterns before deciding on an approach.
---

# Research: Architecture Pattern

## Purpose
Identify proven patterns that solve the user's structural problem, explain their trade-offs with evidence from real-world deployments, and recommend the best fit for the project's current scale and constraints.

## Pattern categories to consider

### Service decomposition
- Monolith → Modular Monolith → Microservices → Nano-services
- Strangler Fig (incremental migration pattern)
- Backend for Frontend (BFF)

### Data patterns
- CQRS (Command Query Responsibility Segregation)
- Event Sourcing
- Saga (distributed transactions)
- Outbox Pattern (reliable event publishing)
- Read Replica / CQRS read models

### Communication patterns
- Request/Response (REST, gRPC, GraphQL)
- Event-driven / Message Queue (Kafka, RabbitMQ, SQS)
- Pub/Sub
- Choreography vs Orchestration (for Sagas)

### Resilience patterns
- Circuit Breaker
- Bulkhead
- Retry with exponential backoff + jitter
- Dead Letter Queue (DLQ)

### Frontend architecture
- MPA vs SPA vs SSR vs ISR
- Islands Architecture
- Micro-frontends

## Investigation steps

1. **Clarify the constraints first** — scale (RPS, data volume), team size, deployment target, consistency requirements, latency budget
2. **Identify candidate patterns** — list 2–4 patterns that could solve the problem
3. **Find real-world examples** — search engineering blogs from companies at similar scale

```bash
# Search for engineering blog posts about the pattern
# (use web search if available, otherwise use known sources)
# Known sources: Netflix Tech Blog, Uber Engineering, Shopify Engineering,
# Martin Fowler's bliki, AWS Architecture Blog, The Pragmatic Engineer
```

4. **Map patterns to project constraints**

| Pattern | Fits when | Breaks when |
|---------|-----------|-------------|
| Microservices | Teams > 10, independent deploy needed | Small team, shared DB, tight latency |
| CQRS | Read/write ratio heavily skewed | Simple CRUD, small team, low complexity tolerance |
| Event Sourcing | Audit trail required, temporal queries | Simple state, no event schema discipline |

## Output format
Each pattern in the deep-dive must include:
- **What it solves** (one sentence)
- **How it works** (diagram or bullet flow)
- **When to use it** (concrete triggers)
- **When NOT to use it** (failure modes, complexity cost)
- **Real-world example** (company + scale + outcome)
- **Implementation complexity**: `Low` / `Medium` / `High` / `Very High`

## Constraints
- Never recommend microservices for a small team without flagging the operational overhead cost explicitly
- Always state the **operational maturity** required (e.g., event sourcing requires event schema governance discipline)
- Patterns are not silver bullets — state what problems remain *unsolved* after adoption
- Prefer battle-tested patterns over novel ones unless the user's constraint truly requires novelty
