---
name: se-error-handling-plan
description: Design and implement a consistent, meaningful error handling strategy for a feature or module including custom error types, error propagation, and user-facing messages. Use this skill whenever the software engineer agent is implementing error handling, designing error responses, or when existing error handling is inconsistent or missing. Trigger on any "error handling", "exception", "try catch", "error response", or "failure case" concern.
---

# Error Handling Plan

Errors should be predictable, informative, and handled at the right layer.

## Error Handling Principles

1. **Fail fast** — validate inputs early, before doing expensive work
2. **Fail loudly in dev, gracefully in prod** — don't swallow errors silently
3. **Handle at the right layer** — don't catch what you can't handle
4. **Never expose internals** — stack traces and DB errors must not reach clients
5. **Be specific** — `UserNotFoundError` beats `Error("something went wrong")`

## Error Categories

### Domain / Business Errors
Predictable failures within business rules:
- `UserNotFoundError`
- `InsufficientBalanceError`
- `DuplicateEmailError`
- `OrderAlreadyCancelledError`

These are **expected** — handle them explicitly, return structured responses.

### Validation Errors
Bad input from the caller:
- Missing required fields
- Invalid format (email, UUID, date)
- Out-of-range values

Catch at the entry point (controller/handler), return `400 Bad Request`.

### Infrastructure Errors
Unexpected failures from external systems:
- DB connection failure
- Third-party API timeout
- File not found

Log with full context, return `500 Internal Server Error` to client (no internals exposed).

### Programming Errors
Bugs — should never be caught:
- `TypeError`, `NullReferenceException`
- Let these propagate, crash loudly, alert on-call

## Error Class Design

```typescript
// Base
class AppError extends Error {
  constructor(
    public readonly message: string,
    public readonly code: string,
    public readonly statusCode: number,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

// Domain
class UserNotFoundError extends AppError {
  constructor(userId: string) {
    super(`User ${userId} not found`, 'USER_NOT_FOUND', 404, { userId });
  }
}

// Validation
class ValidationError extends AppError {
  constructor(fields: Record<string, string>) {
    super('Validation failed', 'VALIDATION_ERROR', 400, { fields });
  }
}
```

## Error Propagation Rules

```
Controller/Handler   ← catches all, formats response
      ↑
   Service          ← catches infrastructure errors, re-throws as domain errors
      ↑
  Repository        ← catches DB errors, wraps into AppError
      ↑
  External I/O      ← raw errors
```

## Error Response Shape

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User 123 not found",
    "details": { "userId": "123" }
  }
}
```

## Output Format

```
## Error Handling Plan for [feature]

### Custom Error Types
[list of error classes with code + statusCode]

### Error Propagation
[which layer catches what]

### Implementation
[full error class code + handler code]

### Logging Strategy
[what gets logged, at what level, with what context]
```
