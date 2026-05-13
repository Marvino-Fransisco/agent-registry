# DDD vs Simple Model

## DDD vs Simple Model Standards

- Domain complexity determines the approach — not team preference
- DDD is a strategic tool, not a default starting point
- A simple model can always evolve into DDD when justified

---

## Instructions

### Step 1 - Identification

> Understand the domain before choosing the modeling approach

- Identify the number of distinct business entities
- Identify whether entities have complex lifecycle rules (state machines, invariants, aggregate roots)
- Identify whether the business language is rich and ubiquitous or simple and flat
- Identify whether multiple bounded contexts exist or could emerge

---

### Step 2 - Evaluate

> Match domain complexity to the appropriate modeling approach

#### Simple Model

- Best for: CRUD-dominant workflows, thin business logic, few validation rules
- Approach: Active Record or plain data models with service-layer validation
- When to use: Entities have no complex invariants, no aggregate boundaries needed
- Characteristics: Model = data shape, logic lives in services

#### DDD (Domain-Driven Design)

- Best for: Rich business rules, complex entity lifecycles, multiple bounded contexts
- Approach: Aggregates, value objects, domain events, repositories, domain services
- When to use: Invariants must be enforced at the entity level, business language is nuanced
- Characteristics: Model = behavior + data, logic lives inside the domain

---

### Step 3 - Decision Checklist

> Answer honestly. If most answers point to "no," use a simple model.

- Does the entity have invariants that must never be violated regardless of where it is modified?
- Are there multiple business rules that depend on the state of an entity?
- Is there a rich business vocabulary that differs from technical terminology?
- Do entities have lifecycle rules (e.g., an order can only be cancelled before shipment)?
- Are there aggregate boundaries where consistency must be guaranteed?

---

### Step 4 - Output

> Provide the modeling recommendation with justification

- State the chosen approach and why
- List the key domain concepts that influenced the decision
- If DDD, define the bounded contexts and aggregate roots
- If simple model, document the boundary where DDD might become necessary

---

## Edge Cases

### 1. Mixed Complexity Within One Project

> Some modules are CRUD while others have rich business rules

**Example:**

- User management is simple CRUD, but Order processing has complex state machines

**Rule of Thumb:**

- Apply DDD only to the modules that need it, keep the rest simple

**Practical Trade-off:**

- Inconsistent modeling styles across modules vs appropriate complexity per module
- Inconsistency is acceptable when each module's modeling matches its actual complexity

---

### 2. Uncertain Domain Complexity

> The domain is not well understood yet and complexity is unclear

**Example:**

- Early-stage startup building a new product category

**Rule of Thumb:**

- Start with a simple model
- Refactor to DDD when invariants and aggregate boundaries become clear

**Practical Trade-off:**

- Possible refactoring cost later vs over-engineering risk now
- Refactoring toward DDD is safer than reversing from DDD to simple

---

## Bad Example

### 1. DDD for a User Profile CRUD

```typescript
class UserAggregateRoot {
  private events: DomainEvent[] = [];

  constructor(private user: UserEntity) {}

  changeName(firstName: string, lastName: string): void {
    this.user.firstName = firstName;
    this.user.lastName = lastName;
    this.events.push(new UserNameChangedEvent(this.user.id));
  }
}
```

For a feature that just saves first name and last name to a database.

### 2. Anemic Model for Order Processing

```typescript
class Order {
  id: string;
  status: string;
  items: OrderItem[];
}

// All logic in service, entity has no protection
orderService.cancel(order);
order.status = "cancelled";
```

---

## Good Example

### 1. Simple Model for User Profile

```typescript
interface User {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
}

class UserService {
  async updateProfile(id: string, data: UpdateUserDTO): Promise<User> {
    return this.userRepo.update(id, data);
  }
}
```

### 2. DDD for Order Processing

```typescript
class Order {
  private status: OrderStatus;
  private items: OrderItem[];

  get isCancellable(): boolean {
    return this.status === OrderStatus.PENDING || this.status === OrderStatus.CONFIRMED;
  }

  cancel(reason: string): void {
    if (!this.isCancellable) {
      throw new DomainError("Order cannot be cancelled in current state");
    }
    this.status = OrderStatus.CANCELLED;
    this.addDomainEvent(new OrderCancelledEvent(this.id, reason));
  }
}
```
