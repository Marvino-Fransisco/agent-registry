# Bulkhead

## Bulkhead Standards

- Isolate critical resources so that failure in one area does not cascade to others
- Each isolated pool must have its own resource limits
- Bulkheads must be applied proactively, not after a failure occurs
- The guiding question: **Is the blast radius of a failure worth the cost of containing it?**

---

## Instructions

### Step 1 - Identification

> Understand whether bulkhead isolation is needed

- Identify whether the system has distinct workloads with different criticality levels
- Identify whether a slow or failing component can consume all available resources
- Identify whether there are shared resource pools (thread pools, connection pools, memory)

---

### Step 2 - Evaluate

> Determine if Bulkhead is the right fit by answering: is the blast radius worth containing?

| Situation | Worth It? | Why |
| --------- | --------- | --- |
| Payment service vs. recommendation service sharing resources | Yes | Very different criticality — payment must stay alive even if recommendations fail |
| Two equally critical, non-degradable services | No | No graceful degradation possible — isolating them does not help when both must succeed |
| High traffic, mixed workload types | Yes | Isolation prevents one workload type from starving others |
| Low traffic, uniform workload | No | Failure scenario unlikely to occur — the overhead is not justified |
| Polyglot, distributed teams | Yes | Teams own their own pools — independent scaling and deployment |
| Single small service, one external dependency | No | Just set a good timeout and circuit breaker |

---

### Step 3 - Heuristic Rule

> Before applying Bulkhead, follow this order

1. **Reach for Circuit Breaker + Timeout first**
2. Only add Bulkhead when you have **evidence** (load testing, incidents) that resource exhaustion in one area is actually starving another

---

### Step 4 - Output

> Provide the bulkhead design

- Define which workloads are isolated
- Define the resource limits per pool (connections, threads, memory, concurrency)
- Define the fallback behavior when a pool is exhausted

---

### Step 5 - Output

> Provide the bulkhead design

- Define which workloads are isolated
- Define the resource limits per pool (connections, threads, memory, concurrency)
- Define the fallback behavior when a pool is exhausted

---

## Edge Cases

### 1. Over-Partitioning

> Creating too many isolated pools wastes resources

**Rule of Thumb:**

- Isolate by criticality level, not by every individual operation
- Typically: critical pool, standard pool, background pool

**Practical Trade-off:**

- More pools = better isolation but higher resource overhead and lower utilization

---

## Bad Example

### 1. Single Connection Pool for All Downstream Calls

```typescript
// All services share one HTTP client with one connection pool
const httpClient = new HttpClient({ maxConnections: 100 });

// A slow payment service exhausts all 100 connections
// User service, order service — all blocked
```

---

## Good Example

### 1. Isolated Pools by Criticality

```typescript
// Critical operations (auth, payments) get their own pool
const criticalClient = new HttpClient({
  maxConnections: 50,
  timeout: 5000,
});

// Standard operations (product listing, search) share a pool
const standardClient = new HttpClient({
  maxConnections: 30,
  timeout: 10000,
});

// Background operations (analytics, emails) get a separate pool
const backgroundClient = new HttpClient({
  maxConnections: 20,
  timeout: 30000,
});
```
