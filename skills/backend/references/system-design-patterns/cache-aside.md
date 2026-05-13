# Cache-Aside

## Cache-Aside Standards

- The application is responsible for reading from and writing to the cache
- Cache is not the source of truth — the database is
- Cache invalidation must be handled explicitly, never assumed to happen automatically

---

## Instructions

### Step 1 - Identification

> Understand whether caching is appropriate

- Identify whether the data is read frequently and updated infrequently
- Identify whether the read latency is a measurable problem
- Identify whether stale data is acceptable for the use case

---

### Step 2 - Evaluate

> Determine if Cache-Aside is the right fit

- Best for: Read-heavy workloads with relatively static data (user profiles, product catalogs, config)
- Best for: Reducing database load under high read traffic
- NOT for: Data that changes frequently and must always be fresh
- NOT for: Write-heavy workloads with few reads

---

### Step 3 - Output

> Provide the caching design

- Define what data is cached
- Define the cache key strategy
- Define the TTL (time-to-live) for each cached item
- Define the invalidation strategy (TTL expiry, explicit invalidation on write, event-driven)

---

## Edge Cases

### 1. Cache Stampede

> Many requests miss the cache simultaneously and all hit the database at once

**Example:**

- Cache expires for a popular product page, 1000 requests hit the database simultaneously

**Rule of Thumb:**

- Use cache locking or request coalescing to prevent stampede
- Alternatively, use early expiration with background refresh

**Practical Trade-off:**

- Slightly more complex caching logic vs potential database overload

---

### 2. Stale Data Tolerance

> Different data types have different freshness requirements

**Example:**

- Product price must be fresh, product description can be stale for minutes

**Rule of Thumb:**

- Set TTL based on the business tolerance for staleness
- Critical data should use explicit invalidation, not just TTL

**Practical Trade-off:**

- Longer TTL = better performance but more stale reads

---

## Bad Example

### 1. No TTL and No Invalidation

```typescript
async function getUser(id: string) {
  const cached = await cache.get(`user:${id}`);
  if (cached) return cached;

  const user = await db.query("SELECT * FROM users WHERE id = $1", [id]);
  await cache.set(`user:${id}`, user); // no TTL — stale forever
  return user;
}
```

### 2. Caching Rapidly Changing Data

```typescript
// Stock price changes every second — caching it makes no sense
async function getStockPrice(symbol: string) {
  const cached = await cache.get(`stock:${symbol}`);
  if (cached) return cached;
  return await fetchStockPrice(symbol);
}
```

---

## Good Example

### 1. Cache-Aside With TTL and Invalidation

```typescript
async function getUser(id: string): Promise<User> {
  const cached = await cache.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  const user = await userRepo.findById(id);
  await cache.set(`user:${id}`, JSON.stringify(user), "EX", 300); // 5 min TTL
  return user;
}

async function updateUser(id: string, data: UpdateUserDTO): Promise<User> {
  const user = await userRepo.update(id, data);
  await cache.del(`user:${id}`); // explicit invalidation on write
  return user;
}
```

### 2. Different TTLs for Different Data Types

```typescript
const CACHE_TTL = {
  userProfile: 300,     // 5 minutes — changes rarely
  productCatalog: 600,  // 10 minutes — changes infrequently
  appConfig: 3600,      // 1 hour — changes very rarely
  searchResults: 60,    // 1 minute — changes more often
};
```
