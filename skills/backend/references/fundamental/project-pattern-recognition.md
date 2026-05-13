# Project Pattern Recognition

## Project Pattern Recognition Standards

- Pattern recognition must be based on actual file and folder structure — never assumed
- Patterns must be documented as-is before any recommendations are made
- Anti-patterns must be flagged explicitly

---

## Instructions

### Step 1 - Identification

> Understand what to look for before reading any files

- Identify what architecture pattern the project claims to follow (README, docs)
- Identify the actual folder structure by listing directories
- Identify the entry points and how the application boots

---

### Step 2 - Scan Project Structure

> Read the actual project layout and map it to known patterns

- Scan the root-level directories and their purpose
- Scan the `src/` directory (or equivalent) for layer separation
- Map each directory to its architectural role (presentation, domain, infrastructure, etc.)
- Identify whether the structure follows a single pattern or is mixed

---

### Step 3 - Classify

> Classify the project into one or more known patterns

- Layered (Controller → Service → Repository)
- Clean Architecture (Entities → Use Cases → Adapters → Frameworks)
- Hexagonal (Domain → Ports → Adapters)
- CQRS (Command/Query separation)
- Flat / No Pattern
- Mixed (multiple patterns in different modules)

---

### Step 4 - Output

> Document the recognized patterns and deviations

- State the primary architecture pattern found
- List any deviations from the claimed pattern
- List any anti-patterns discovered
- Provide a summary of the project's structural decisions

---

## Edge Cases

### 1. Partially Migrated Architecture

> The project started with one pattern and is migrating to another

**Example:**

- Legacy modules use flat structure, new modules follow Hexagonal

**Rule of Thumb:**

- Document both patterns and the migration boundary
- Note which pattern is the target state

**Practical Trade-off:**

- Inconsistency is expected during migration — flag it but do not recommend a rollback

---

### 2. Framework-Imposed Structure

> The framework dictates the folder structure (e.g., Next.js, NestJS, Rails)

**Example:**

- NestJS enforces module/controller/service structure

**Rule of Thumb:**

- Recognize the framework's conventions as the primary pattern
- Evaluate whether the project adds any custom layers on top

**Practical Trade-off:**

- Framework conventions are usually well-tested — follow them unless there is a strong reason to deviate

---

## Bad Example

### 1. Claiming Clean Architecture Without Dependency Inversion

```
src/
  entities/
    user.entity.ts        # imports TypeORM decorator
  use-cases/
    create-user.ts        # imports repository directly from infrastructure
  infrastructure/
    user.repository.ts
```

Entities depend on infrastructure. Not Clean Architecture.

---

## Good Example

### 1. Honest Recognition

> The project claims Hexagonal Architecture. After scanning:

```
src/
  domain/
    user/
      user.ts                    # pure domain, no framework imports
  ports/
    user-repository.port.ts      # interface only
  adapters/
    postgres/
      user-postgres.adapter.ts   # implements port
  app.ts
```

Result: Hexagonal Architecture, correctly applied. Domain has zero infrastructure imports.
