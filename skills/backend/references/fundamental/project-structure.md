# Project Structure

## Project Structure Standards

- Architecture choice must match project complexity — never over-engineer a simple service
- Architecture must be decided before writing any code
- The chosen pattern must be consistently applied across the entire project

---

## Instructions

### Step 1 - Identification

> Understand the scope and complexity of the project before choosing an architecture

- Identify the project type (monolith, microservice, serverless, CLI tool)
- Identify the expected scale (small team prototype, enterprise production system)
- Identify the domain complexity (CRUD app, complex business logic, event-driven)
- Identify whether the project will have multiple read/write models

---

### Step 2 - Evaluate Architecture Options

> Match the project needs to the appropriate architecture pattern

#### CSR (Call, Store, Return) / Simple Layered

- Best for: Simple CRUD services, small teams, prototypes
- Structure: Controller → Service → Repository → Database
- When to use: Domain logic is thin, few business rules

#### Clean Architecture

- Best for: Medium to large projects with clear business rules
- Structure: Entities → Use Cases → Interface Adapters → Frameworks & Drivers
- When to use: Business logic is complex and must be independent of frameworks
- Key rule: Dependencies point inward only

#### Hexagonal Architecture (Ports & Adapters)

- Best for: Projects that need to swap external systems (databases, APIs, queues)
- Structure: Domain → Ports (inbound/outbound) → Adapters
- When to use: Multiple integrations, testing isolation is critical
- Key rule: Domain has zero dependencies on infrastructure

#### CQRS

- Best for: Projects with distinct read vs write workloads
- Structure: Command side (writes) separate from Query side (reads)
- When to use: Read and write models differ significantly in shape or scale
- Key rule: Only use when the read/write separation solves a real problem

---

### Step 3 - Output

> Provide the architecture recommendation with justification

- State which architecture was chosen and why
- Provide the folder structure
- Identify any patterns that should be combined (e.g., Hexagonal + CQRS)

---

## Edge Cases

### 1. Multiple Architecture Patterns Needed

> A single project may benefit from combining patterns

**Example:**

- Hexagonal Architecture for the core domain + CQRS for the reporting subsystem

**Rule of Thumb:**

- Combine patterns only when each pattern solves a distinct problem
- Never combine patterns just because they "look good together"

**Practical Trade-off:**

- More patterns = more structure = more onboarding cost
- Only combine if the complexity justifies it

---

### 2. Starting Simple Then Migrating

> Choosing a simple architecture with a plan to migrate later

**Example:**

- Start with layered architecture, migrate to Hexagonal when integrations grow

**Rule of Thumb:**

- Start simple when the domain is not yet well understood
- Migration path should be documented from day one

**Practical Trade-off:**

- Faster initial delivery vs potential refactoring cost later

---

## Bad Example

### 1. Clean Architecture for a Simple CRUD API

```
src/
  domain/
    entities/
    value-objects/
    use-cases/
  application/
    services/
    dto/
  infrastructure/
    repositories/
    external/
  presentation/
    controllers/
    middleware/
```

For a service that just reads and writes to a single table.

### 2. Flat Structure for a Complex Domain

```
src/
  index.ts
  handlers.ts
  database.ts
  utils.ts
```

For a payment processing system with multiple business rules.

---

## Good Example

### 1. Layered Architecture for Simple CRUD

```
src/
  controllers/
    user.controller.ts
  services/
    user.service.ts
  repositories/
    user.repository.ts
  models/
    user.model.ts
  config/
    database.ts
  middlewares/
    auth.middleware.ts
  app.ts
```

### 2. Hexagonal Architecture for Integration-Heavy Service

```
src/
  domain/
    user/
      user.entity.ts
      user.repository.port.ts
  application/
    user/
      create-user.use-case.ts
      get-user.use-case.ts
  ports/
    inbound/
      user.controller.ts
    outbound/
      user.repository.port.ts
  adapters/
    inbound/
      rest/
        user-rest.controller.ts
    outbound/
      postgres/
        user-postgres.repository.ts
      redis/
        user-cache.adapter.ts
  config/
    database.ts
  app.ts
```
