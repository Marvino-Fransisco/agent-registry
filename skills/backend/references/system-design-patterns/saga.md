# Saga

## Saga Standards

- A saga manages a sequence of local transactions across services
- Each local transaction publishes an event or triggers the next step
- If a step fails, the saga executes compensating transactions for all completed steps
- Two implementations: Choreography-based (events) and Orchestration-based (central coordinator)

---

## Instructions

### Step 1 - Identification

> Understand whether a saga is needed

- Identify whether a business process spans multiple services
- Identify whether each service maintains its own database
- Identify whether partial failures must be rolled back
- Identify whether the process has more than 3 steps (fewer steps may use simple compensating transactions)

---

### Step 2 - Evaluate

> Determine if Saga is the right fit and choose the type

#### Choreography-Based Saga

- Best for: Simple sagas with 2-4 steps
- Best for: When services should remain loosely coupled
- Approach: Each service emits events that trigger the next step
- Downside: Hard to trace the full flow, risk of cyclic dependencies

#### Orchestration-Based Saga

- Best for: Complex sagas with many steps, conditional branching, or parallel execution
- Best for: When centralized control and visibility are important
- Approach: A saga orchestrator tells each service what to do
- Downside: Orchestrator can become a single point of control

---

### Step 3 - Output

> Provide the saga design

- Define each step in the saga (service, action, compensating action)
- Define the saga type (choreography or orchestration)
- Define the communication mechanism (events or commands)
- Define the failure handling and compensation flow
- Define idempotency requirements for each step

---

## Edge Cases

### 1. Compensation Ordering

> Compensations must execute in reverse order of the completed steps

**Rule of Thumb:**

- Track the completed steps in order
- On failure, compensate from the last completed step backward

**Practical Trade-off:**

- Reverse-order compensation is simpler to reason about
- Parallel compensation is faster but harder to debug

---

### 2. Saga Stuck in Partial State

> A step fails and compensation also fails

**Rule of Thumb:**

- Retry compensations with exponential backoff
- Alert operations team if compensation fails after max retries
- Never leave a saga in a partial state without alerting

**Practical Trade-off:**

- Manual intervention for stuck sagas is acceptable as a last resort
- Automatic recovery is ideal but complex

---

## Bad Example

### 1. Saga Without Compensation

```typescript
// Order → Reserve Inventory → Charge Payment → Ship
// If Shipping fails: inventory reserved, payment charged, no rollback
```

### 2. Orchestration Saga for 2 Steps

> Over-engineering: a 2-step process does not need a saga orchestrator. Use a simple compensating transaction.

---

## Good Example

### 1. Orchestration-Based Saga for Order Processing

```typescript
const orderSaga = defineSaga("place-order", {
  steps: [
    {
      name: "reserve-inventory",
      action: (ctx) => inventoryService.reserve(ctx.order.items),
      compensate: (ctx) => inventoryService.release(ctx.reservationId),
    },
    {
      name: "charge-payment",
      action: (ctx) => paymentService.charge(ctx.order.payment, ctx.order.total),
      compensate: (ctx) => paymentService.refund(ctx.chargeId),
    },
    {
      name: "schedule-shipping",
      action: (ctx) => shippingService.schedule(ctx.order, ctx.reservationId),
      compensate: (ctx) => shippingService.cancel(ctx.shipmentId),
    },
  ],
  onFailure: "compensate-all",
});
```

### 2. Choreography-Based Saga

```markdown
Order Service: Create Order → emit OrderCreated
  ↓
Inventory Service: Reserve Stock → emit StockReserved
  ↓
Payment Service: Charge Payment → emit PaymentCharged
  ↓
Shipping Service: Schedule Shipment → emit ShipmentScheduled
  ↓
Order Service: Mark Order as Complete → emit OrderCompleted

// If Payment Service emits PaymentFailed:
//   Inventory Service listens → releases stock
//   Order Service listens → marks order as failed
```
