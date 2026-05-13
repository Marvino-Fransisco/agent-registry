# Choreography

## Choreography Standards

- Each service decides independently when to react to events
- There is no central coordinator — services communicate through events
- Each service must be idempotent to handle duplicate events safely

---

## Instructions

### Step 1 - Identification

> Understand whether choreography is appropriate

- Identify whether the workflow involves multiple independent services
- Identify whether the workflow is loosely coupled (services can act independently)
- Identify whether a central orchestrator would add unnecessary coupling

---

### Step 2 - Evaluate

> Determine if Choreography is the right fit

- Best for: Event-driven workflows where each step is independent
- Best for: Systems where services should not know about each other
- NOT for: Workflows that require strict ordering and centralized decision-making
- NOT for: Complex workflows with conditional branching and compensating actions

---

### Step 3 - Output

> Provide the choreography design

- Define the events and their producers/consumers
- Define the event schema for each event
- Define the eventual consistency boundaries

---

## Edge Cases

### 1. Debugging Distributed Workflows

> When something goes wrong, it is hard to trace the flow across services

**Rule of Thumb:**

- Use correlation IDs to trace events across services
- Log every event produced and consumed with the correlation ID

**Practical Trade-off:**

- Simpler services vs harder debugging
- Invest in observability (distributed tracing, event logs)

---

### 2. Event Ordering

> Events may arrive out of order

**Rule of Thumb:**

- Design consumers to be order-agnostic when possible
- Use timestamps or sequence numbers when order matters

**Practical Trade-off:**

- Handling out-of-order events adds complexity to each consumer
- Consider orchestration if strict ordering is critical

---

## Bad Example

### 1. Choreography for a Complex Workflow With Compensation

> A payment workflow with refund-on-failure logic spread across 5 services via events — impossible to trace and debug

---

## Good Example

### 1. Simple Choreography for Decoupled Reactions

```
Order Service → publishes OrderCreated event
  → Inventory Service → reserves stock
  → Notification Service → sends confirmation email
  → Analytics Service → records order event
```

Each service reacts independently. No service knows about the others.
