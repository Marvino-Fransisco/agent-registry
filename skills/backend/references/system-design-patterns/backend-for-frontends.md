# Backend for Frontends (BFF)

## Backend for Frontends Standards

- Each frontend type (web, mobile, IoT) should have its own dedicated backend if their data needs differ significantly
- A BFF must not become a general-purpose API — it serves one frontend type
- A BFF must aggregate, trim, and transform data for its specific frontend, not add business logic

---

## Instructions

### Step 1 - Identification

> Understand whether a BFF is needed

- Identify the number of frontend types consuming the API
- Identify whether different frontends need significantly different response shapes
- Identify whether mobile needs smaller payloads than web
- Identify whether frontends are making multiple calls to assemble a single view

---

### Step 2 - Evaluate

> Determine if BFF is the right fit

- Best for: Web + mobile + IoT each needing different data shapes and sizes
- Best for: Reducing number of round-trips from frontend to backend
- NOT for: Single frontend type (just use a regular API)
- NOT for: When all frontends can consume the same response shape

---

### Step 3 - Output

> Provide the BFF design

- Define which BFFs are needed (e.g., web-bff, mobile-bff)
- Define what each BFF aggregates or transforms
- Define the downstream services each BFF calls

---

## Edge Cases

### 1. BFF Becomes a God Service

> The BFF accumulates business logic that belongs in downstream services

**Rule of Thumb:**

- BFF should only aggregate, filter, and transform — never make business decisions

**Practical Trade-off:**

- Some logic in BFF is acceptable for frontend-specific formatting
- Business rules must stay in domain services

---

## Bad Example

### 1. BFF With Business Logic

```typescript
// mobile-bff applies pricing discounts — this belongs in the order service
app.get("/products", async (req, res) => {
  const products = await productService.getAll();
  const discounted = products.map((p) => ({
    ...p,
    price: p.price * 0.9, // 10% mobile discount — business logic
  }));
  res.json(discounted);
});
```

---

## Good Example

### 1. BFF as Pure Aggregation and Trimming

```typescript
// mobile-bff — smaller payload, aggregated from multiple services
app.get("/dashboard", async (req, res) => {
  const [user, orders, notifications] = await Promise.all([
    userService.getProfile(req.userId),
    orderService.getRecent(req.userId, { limit: 3 }),
    notificationService.getUnread(req.userId, { limit: 5 }),
  ]);

  res.json({
    name: user.firstName,                // trimmed for mobile
    recentOrders: orders.map(serialize), // minimal fields
    unreadCount: notifications.length,   // count only
  });
});
```
