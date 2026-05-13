# CQRS (Command Query Responsibility Segregation)

## CQRS Standards

- Separate write operations (commands) from read operations (queries)
- Commands must not return data — they change state
- Queries must not change state — they return data
- Only use CQRS when the read and write models genuinely differ

---

## Instructions

### Step 1 - Identification

> Understand whether CQRS is justified

- Identify whether the read workload differs significantly from the write workload in shape, scale, or optimization
- Identify whether a single model cannot efficiently serve both reads and writes
- Identify whether different data stores would benefit reads vs writes

---

### Step 2 - Evaluate

> Determine if CQRS is the right fit

- Best for: High-read-low-write systems with complex read projections (dashboards, reporting, search)
- Best for: Event-sourced systems where the write model is the event stream
- NOT for: Simple CRUD applications where one model serves both reads and writes
- NOT for: When the added complexity of synchronization is not justified by the benefit

---

### Step 3 - Output

> Provide the CQRS design

- Define the command model (write side) — entities, aggregates, validation
- Define the query model (read side) — projections, denormalized views, indexes
- Define the synchronization mechanism (events, change data capture, triggers)
- Define the eventual consistency window and how to communicate it to users

---

## Edge Cases

### 1. Eventual Consistency UX

> The read model may be stale after a write

**Example:**

- User updates their profile but the profile page shows old data for a few seconds

**Rule of Thumb:**

- Communicate eventual consistency to the user when relevant
- Use optimistic UI updates when possible

**Practical Trade-off:**

- Stale reads vs complex synchronization
- Most systems tolerate seconds of staleness for the performance benefit

---

### 2. Over-Engineering With CQRS

> Applying CQRS to a simple CRUD app

**Rule of Thumb:**

- If you can use a single database table for both reads and writes efficiently, do not use CQRS

**Practical Trade-off:**

- CQRS adds significant complexity (two models, synchronization, consistency handling)
- Only justified when the performance or scalability benefit is measurable

---

## Bad Example

### 1. CQRS for a Simple Blog

```markdown
// Write side: PostWriteRepository → writes to posts_write table
// Read side: PostReadRepository → reads from posts_read table
// Synchronization: event bus copies data between tables

// All of this for a blog with 100 daily readers and 5 daily posts
// A single posts table would work perfectly
```

---

## Good Example

### 1. CQRS for an E-Commerce Platform

```
// Write side (Command)
// - Order aggregate with business rules, validation, state machine
// - Stored in normalized tables optimized for transactional integrity
// - Emits OrderCreated, OrderCancelled events

// Read side (Query)
// - Order summary projection: denormalized table with customer name, item count, total
// - Customer order history: pre-computed per-customer view
// - Dashboard analytics: aggregated metrics
// - Stored with indexes optimized for specific query patterns

// Synchronization
// - Events from write side → update read projections asynchronously
// - Eventual consistency: ~1 second delay
```
