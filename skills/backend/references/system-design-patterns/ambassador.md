# Ambassador

## Ambassador Standards

- The Ambassador pattern must only be used when you need to offload cross-cutting concerns from the application
- The ambassador must be transparent to the client — the client should not need to know it exists

---

## Instructions

### Step 1 - Identification

> Understand whether the Ambassador pattern is appropriate

- Identify whether there are cross-cutting networking concerns (retry, circuit breaking, logging, security) that need to be handled consistently
- Identify whether these concerns should be offloaded from the application code
- Identify whether you are dealing with a client-side or service-side proxy scenario

---

### Step 2 - Evaluate

> Determine if Ambassador is the right fit

- Best for: Offloading retry logic, connection pooling, monitoring, and security from application code
- Best for: Legacy service integration where you cannot modify the service itself
- NOT for: Simple one-to-one communication without cross-cutting concerns
- NOT for: When the application can handle the concern natively with minimal code

---

### Step 3 - Output

> Provide the pattern recommendation

- State whether the Ambassador pattern should be applied
- Define what concerns the ambassador will handle
- Define the placement (sidecar, standalone proxy, service mesh)

---

## Edge Cases

### 1. Overhead vs Benefit

> Adding an ambassador adds latency and operational complexity

**Rule of Thumb:**

- Use when the cross-cutting concerns are complex enough that embedding them in every service is worse

**Practical Trade-off:**

- Extra hop and infrastructure vs consistent cross-cutting behavior across all services

---

## Bad Example

### 1. Ambassador for a Single Internal Call

> Deploying a sidecar proxy for a service that makes one internal call with no cross-cutting needs

---

## Good Example

### 1. Ambassador as API Gateway for External Clients

> External clients → Ambassador (rate limiting, auth, logging, request transformation) → Internal services
