# Code Pattern Recognition

## Code Pattern Recognition Standards

- Code patterns must be recognized by reading actual source files — never assumed
- Both positive and negative patterns must be documented
- Pattern recognition focuses on how code is written, not what the project structure looks like

---

## Instructions

### Step 1 - Identification

> Understand what code-level patterns to look for

- Identify the programming language and version
- Identify the framework and its conventions
- Identify the testing framework (if any)

---

### Step 2 - Scan Code Patterns

> Read representative source files and identify recurring patterns

- Error handling pattern (try-catch, Result monad, error middleware, exception hierarchy)
- Validation pattern (where validation happens, how errors are reported)
- Dependency injection pattern (constructor, property, module-level)
- Data access pattern (ORM, query builder, raw SQL, repository)
- Response shaping pattern (DTOs, serializers, raw objects)
- Authentication/authorization pattern (middleware, decorators, guards)
- Logging pattern (structured, unstructured, what is logged)
- Async pattern (Promises, async/await, callbacks)

---

### Step 3 - Classify

> Classify the code patterns found

- Consistent: Same pattern applied across all modules
- Partially consistent: Same pattern in most modules, exceptions exist
- Inconsistent: Different patterns for the same concern in different modules
- Absent: No pattern found for a given concern (e.g., no error handling)

---

### Step 4 - Output

> Document all recognized code patterns

- For each concern, state the pattern found and where it is applied
- Flag inconsistencies between modules
- Flag absent patterns that should exist
- Rate the overall code consistency (consistent, partially consistent, inconsistent)

---

## Edge Cases

### 1. Implicit Patterns

> Patterns that exist but are not explicitly defined — team convention without documentation

**Example:**

- Every service method returns `{ success, data, error }` but this is never documented

**Rule of Thumb:**

- Document implicit patterns and recommend formalizing them

**Practical Trade-off:**

- Implicit patterns work for small teams but break down during onboarding

---

### 2. Legacy Code With No Patterns

> Older parts of the codebase that have no discernible pattern

**Example:**

- Legacy endpoint with no error handling, no validation, inline SQL, and no tests

**Rule of Thumb:**

- Document as "no pattern" and flag for potential refactoring
- Do not recommend immediate refactoring unless it blocks current work

**Practical Trade-off:**

- Leave legacy alone if it works and is not actively modified
- Flag it as technical debt for future planning

---

## Bad Example

### 1. Inconsistent Error Handling Across Modules

```typescript
// module-a/user.controller.ts
app.get("/users/:id", async (req, res) => {
  const user = await userService.getUser(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
});

// module-b/order.controller.ts
app.get("/orders/:id", async (req, res) => {
  try {
    const order = await orderService.getOrder(req.params.id);
    res.json(order);
  } catch (e) {
    res.status(500).json({ message: e.message, stack: e.stack });
  }
});

// module-c/product.controller.ts
app.get("/products/:id", async (req, res) => {
  const result = await productService.getProduct(req.params.id);
  res.json(result); // no error handling at all
});
```

Three different error handling patterns in three controllers.

---

## Good Example

### 1. Consistent Error Handling Pattern

```typescript
// shared/error-handler.ts
class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number,
    public code: string
  ) {
    super(message);
  }
}

// middleware/error.middleware.ts
function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { code: err.code, message: err.message },
    });
  }
  logger.error("Unhandled error", err);
  res.status(500).json({ error: { code: "INTERNAL_ERROR", message: "Internal server error" } });
}

// applied consistently in all controllers
app.get("/users/:id", async (req, res, next) => {
  const user = await userService.getUser(req.params.id);
  if (!user) throw new AppError("User not found", 404, "USER_NOT_FOUND");
  res.json(user);
});
```
