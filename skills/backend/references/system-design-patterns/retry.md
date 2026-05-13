# Retry

## Retry Standards

- Retries must only be applied to transient failures (network timeout, service temporarily unavailable)
- Retries must use exponential backoff with jitter
- Retries must have a maximum attempt limit
- Retries must not be applied to non-transient failures (400 Bad Request, 401 Unauthorized, 404 Not Found)

---

## Instructions

### Step 1 - Identification

> Understand whether retry is appropriate

- Identify the type of failure (transient vs permanent)
- Identify whether the operation is idempotent (safe to retry)
- Identify whether the downstream service is designed to handle retries

---

### Step 2 - Evaluate

> Determine if Retry is the right fit

- Best for: Network timeouts, 503 Service Unavailable, 429 Too Many Requests, connection reset
- Best for: Idempotent operations (GET requests, idempotent POSTs with idempotency keys)
- NOT for: Non-idempotent operations without idempotency keys
- NOT for: Permanent failures (400, 401, 403, 404)

---

### Step 3 - Output

> Provide the retry design

- Define which operations are retried
- Define the retry strategy (max attempts, backoff, jitter)
- Define which errors trigger retries vs fail immediately
- Define the circuit breaker integration (stop retrying when circuit is open)

---

## Edge Cases

### 1. Non-Idempotent Operations

> Retrying a payment charge could result in double-charging

**Rule of Thumb:**

- Use idempotency keys for POST/PUT operations
- The downstream service must recognize and safely handle duplicate requests

**Practical Trade-off:**

- Idempotency requires the downstream service to track request IDs
- Without idempotency, do not retry non-idempotent operations

---

### 2. Retry Storms

> Many clients retrying simultaneously can overwhelm the recovering service

**Rule of Thumb:**

- Always add jitter to backoff intervals
- Use exponential backoff to space out retries

**Practical Trade-off:**

- Jitter adds randomness but prevents synchronized retry bursts

---

## Bad Example

### 1. Immediate Retry Without Backoff

```typescript
async function callService(url: string) {
  try {
    return await fetch(url);
  } catch {
    return await fetch(url); // immediate retry — no backoff, no max attempts
  }
}
```

### 2. Retrying a 404

```typescript
// 404 is permanent — retrying will never succeed
if (response.status === 404) {
  await retry(() => fetch(url)); // wasted effort
}
```

---

## Good Example

### 1. Exponential Backoff With Jitter

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  options: { maxAttempts: number; baseDelay: number }
): Promise<T> {
  const RETRYABLE_STATUS = new Set([408, 429, 500, 502, 503, 504]);

  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (!isRetryable(error, RETRYABLE_STATUS) || attempt === options.maxAttempts) {
        throw error;
      }
      const delay = Math.min(
        options.baseDelay * Math.pow(2, attempt - 1) + Math.random() * 100,
        30000
      );
      await sleep(delay);
    }
  }
  throw new Error("Max retry attempts reached");
}
```
