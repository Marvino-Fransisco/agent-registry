# Coding Patterns

## Coding Patterns Standards

- One file must have a single clear responsibility
- File naming must reflect its content
- Consistency across the project is non-negotiable — pick one pattern and apply it everywhere

---

## Instructions

### Step 1 - Identification

> Understand the team and project constraints before choosing a coding pattern

- Identify the team size and experience level
- Identify whether the project uses dependency injection
- Identify the testing strategy (unit tests, integration tests, both)
- Identify the framework conventions (if any)

---

### Step 2 - Evaluate

> Match the project needs to the appropriate coding pattern

#### Pattern A — One File One Function

- Best for: Functional programming, utility libraries, AWS Lambda handlers, small focused modules
- Approach: Each file exports exactly one primary function or class
- When to use: Maximum testability, minimal coupling, tree-shaking friendly
- File naming: `get-user.ts`, `create-order.ts`, `validate-email.ts`

#### Pattern B — One File One Context (Multiple Related Functions)

- Best for: Related operations that share private state or helpers, service classes, repository classes
- Approach: Each file groups related functions or methods under a single context
- When to use: Functions share dependencies, private helpers, or form a cohesive unit
- File naming: `user-service.ts`, `order-repository.ts`, `auth-controller.ts`

---

### Step 3 - Decision Rules

> Rules for when to use which pattern

- Use Pattern A when the function has no shared state with other functions
- Use Pattern B when functions within the context share private helpers or state
- Never mix Pattern A and Pattern B for the same type of module
- A context file must not exceed a reasonable size — if it does, split into Pattern A

---

### Step 4 - Output

> Provide the coding pattern recommendation with examples

- State the chosen pattern and why
- Provide file structure examples
- Provide naming conventions

---

## Edge Cases

### 1. Utility Functions That Grow Into a Service

> A single utility file that accumulates related functions over time

**Example:**

- `format-date.ts` grows into `format-date.ts`, `parse-date.ts`, `validate-date.ts`, all importing shared constants

**Rule of Thumb:**

- When 3+ files in the same domain share private helpers, consolidate into a context file

**Practical Trade-off:**

- More files to manage vs clearer single responsibility
- Consolidate when the cognitive overhead of multiple files exceeds the benefit

---

### 2. Testing Implications

> The chosen pattern affects how tests are structured

**Example:**

- Pattern A: Each file is trivially testable in isolation
- Pattern B: Private helpers may need to be exposed for testing or tested indirectly

**Rule of Thumb:**

- If testability is the highest priority, lean toward Pattern A

**Practical Trade-off:**

- Pattern A has more files and imports to manage but cleaner test boundaries
- Pattern B has fewer files but may require more setup in tests

---

## Bad Example

### 1. Mixing Patterns Without Consistency

```
src/
  services/
    get-user.ts
    user-service.ts
    create-order.ts
    order.ts
```

### 2. God File (One Context With Too Many Responsibilities)

```typescript
// user-service.ts — handles everything
export function getUser() {}
export function createUser() {}
export function sendWelcomeEmail() {}
export function generateJWT() {}
export function validatePassword() {}
export function hashPassword() {}
```

---

## Good Example

### 1. Pattern A — One File One Function

```
src/
  handlers/
    get-user.ts
    create-user.ts
    update-user.ts
    delete-user.ts
```

### 2. Pattern B — One File One Context

```
src/
  services/
    user-service.ts
    order-service.ts
    auth-service.ts
```

```typescript
// user-service.ts
class UserService {
  constructor(
    private userRepo: UserRepository,
    private hashService: HashService
  ) {}

  async getUser(id: string): Promise<User> { /* ... */ }
  async createUser(data: CreateUserDTO): Promise<User> { /* ... */ }
  async updateUser(id: string, data: UpdateUserDTO): Promise<User> { /* ... */ }
}
```

### 3. Pattern B With Clear Boundaries

```typescript
// auth-service.ts — only auth-related logic
class AuthService {
  async login(email: string, password: string): Promise<Token> { /* ... */ }
  async logout(token: string): Promise<void> { /* ... */ }
  async refreshToken(token: string): Promise<Token> { /* ... */ }
}
```
