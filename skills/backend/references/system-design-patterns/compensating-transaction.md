# Compensating Transaction

## Compensating Transaction Standards

- Use when a distributed transaction spans multiple services and cannot use a single database transaction
- Every operation must have a corresponding compensating (undo) operation
- Compensation must be idempotent — it must be safe to call multiple times

---

## Instructions

### Step 1 - Identification

> Understand whether compensating transactions are needed

- Identify whether an operation spans multiple services or databases
- Identify whether atomic transactions (ACID) are not possible across the boundary
- Identify whether partial failures need to be rolled back

---

### Step 2 - Evaluate

> Determine if Compensating Transaction is the right fit

- Best for: Multi-service operations where each service owns its own data
- Best for: Financial transactions, order processing, booking systems
- NOT for: Operations within a single database (use regular transactions)
- NOT for: Operations where partial failure is acceptable (eventual consistency)

---

### Step 3 - Output

> Provide the compensating transaction design

- Define each step in the distributed operation
- Define the compensating action for each step
- Define the execution order (forward and reverse)
- Define the failure handling for compensations themselves

---

## Edge Cases

### 1. Compensation Failure

> What happens when the compensating action itself fails?

**Rule of Thumb:**

- Compensations must be retried (at-least-once execution)
- Log and alert when a compensation fails for manual intervention
- Design compensations to be idempotent so retries are safe

**Practical Trade-off:**

- Idempotent compensation requires extra tracking but guarantees safety

---

### 2. Non-Reversible Actions

> Some actions cannot be truly undone (email sent, SMS delivered)

**Rule of Thumb:**

- Use semantic compensation instead of literal undo
- Example: Cannot unsend email, but can send a follow-up cancellation email

**Practical Trade-off:**

- Semantic compensation may not fully restore the previous state
- Document the residual effects clearly

---

## Bad Example

### 1. No Compensation for Multi-Service Operation

```typescript
async function placeOrder(order: Order) {
  await inventoryService.reserve(order.items);        // step 1
  await paymentService.charge(order.paymentMethod);    // step 2
  await shippingService.schedule(order);               // step 3
  // If step 3 fails: inventory is reserved, payment is charged, nothing is rolled back
}
```

---

## Good Example

### 1. Compensating Transaction With Rollback

```typescript
async function placeOrder(order: Order) {
  const compensationStack: Compensation[] = [];

  try {
    // Step 1: Reserve inventory
    const reservation = await inventoryService.reserve(order.items);
    compensationStack.push(() => inventoryService.release(reservation.id));

    // Step 2: Charge payment
    const charge = await paymentService.charge(order.paymentMethod, order.total);
    compensationStack.push(() => paymentService.refund(charge.id));

    // Step 3: Schedule shipping
    const shipment = await shippingService.schedule(order);
    compensationStack.push(() => shippingService.cancel(shipment.id));

  } catch (error) {
    // Execute compensations in reverse order
    for (const compensate of compensationStack.reverse()) {
      try {
        await compensate();
      } catch (compensationError) {
        logger.error("Compensation failed", { compensationError });
        alertOpsTeam(compensationError);
      }
    }
    throw error;
  }
}
```
