# Circuit Breaker

## Circuit Breaker Standards

- A circuit breaker must protect against cascading failures from downstream dependencies
- The circuit breaker must have three states: Closed, Open, Half-Open
- The circuit breaker must fail fast when open — do not attempt the call

---

## Instructions

### Step 1 - Identification

> Understand whether a circuit breaker is needed

- Identify whether the system calls external services or dependencies that can fail
- Identify whether a failing downstream service can cause the calling service to become unresponsive
- Identify whether timeouts alone are insufficient (resource exhaustion from waiting)

---

### Step 2 - Evaluate

> Determine if Circuit Breaker is the right fit

- Best for: External API calls, database connections, third-party integrations
- Best for: Preventing thread/connection pool exhaustion from slow dependencies
- NOT for: Internal calls that are highly reliable and fast
- NOT for: When a simple timeout is sufficient

---

### Step 3 - Output

> Provide the circuit breaker design

- Define which downstream calls are protected
- Define the failure threshold (number of failures before opening)
- Define the open duration (how long before attempting half-open)
- Define the fallback behavior when the circuit is open

---

## Edge Cases

### 1. Fallback Quality

> What should happen when the circuit is open?

**Example:**

- Return cached data, default value, or a graceful error

**Rule of Thumb:**

- Every circuit breaker must have a defined fallback
- The fallback must be meaningful, not just "service unavailable"

**Practical Trade-off:**

- Stale cached data vs complete failure
- Stale data is almost always better than a cascading outage

---

### 2. Tuning Thresholds

> Failure threshold and timeout values are critical

**Rule of Thumb:**

- Start conservative (5 failures in 60 seconds) and adjust based on production behavior
- Monitor circuit breaker state changes as a key metric

**Practical Trade-off:**

- Too sensitive: circuit opens too often, reducing availability
- Too lenient: circuit opens too late, allowing cascading failures

---

## Bad Example

### 1. Circuit Breaker Without Fallback

```typescript
const breaker = new CircuitBreaker();

async function getInventory(productId: string) {
  return breaker.execute(() => inventoryService.getStock(productId));
  // When circuit is open: throws error with no graceful handling
}
```

---

## Good Example

### 1. Circuit Breaker With Fallback

```typescript
const breaker = new CircuitBreaker({
  failureThreshold: 5,
  resetTimeout: 30000, // 30 seconds before half-open
});

async function getInventory(productId: string) {
  return breaker.execute(
    () => inventoryService.getStock(productId),
    () => cache.get(`inventory:${productId}`) // fallback to cache
  );
}
```
