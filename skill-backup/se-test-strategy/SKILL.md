---
name: se-test-strategy
description: Design and implement a comprehensive test strategy covering unit, integration, and edge case tests for a given feature or module. Use this skill whenever the software engineer agent is writing or planning tests. Trigger on any "write tests", "test this", "test strategy", "coverage", or "how do I test" request.
---

# Test Strategy

Design tests that are meaningful, maintainable, and fast.

## Test Pyramid

```
        [E2E]           ← Few, slow, high confidence
      [Integration]     ← Some, medium speed
    [Unit Tests]        ← Many, fast, isolated
```

Write mostly unit tests, some integration tests, very few E2E tests.

## Test Types

### Unit Tests
- Test a single function/method in isolation
- Mock all external dependencies (DB, API, filesystem)
- Should be fast (<10ms each)
- Cover: happy path, edge cases, error cases

### Integration Tests
- Test how two or more modules work together
- Use real dependencies or close approximations (test DB, in-memory store)
- Cover: data flows, module boundaries, contract adherence

### Edge Case Tests
Always test:
- Empty / null / undefined inputs
- Boundary values (0, -1, max int, empty string)
- Concurrent access (if applicable)
- Large payloads
- Invalid types or malformed data

## Test Design Process

### 1. Identify test scope
- What is the unit under test?
- What are its dependencies?
- What are its outputs/side effects?

### 2. Define test cases
For each function/method:
```
Given [state/input]
When  [action]
Then  [expected output/side effect]
```

### 3. Identify mocks needed
- External services → mock/stub
- Database → in-memory or test DB
- Time/randomness → inject via dependency

### 4. Write tests first (TDD) or after — either way:
- Test names must describe behaviour, not implementation
- One assertion concept per test
- Tests must be independent (no shared mutable state)

## Naming Convention
```
[unit]_[scenario]_[expected result]

e.g.:
createUser_withDuplicateEmail_throwsConflictError
calculateDiscount_forVIPUser_returns20Percent
processPayment_whenGatewayTimesOut_retriesThreeTimes
```

## Coverage Targets
- Business logic: **>90%**
- Utility/helpers: **>80%**
- Controllers/handlers: **>70%**
- Infrastructure/adapters: **>60%**

## Output Format

```
## Test Plan for [feature/module]

### Unit Tests
- [ ] [test name] — [what it validates]

### Integration Tests
- [ ] [test name] — [what it validates]

### Edge Cases
- [ ] [test name] — [what it validates]

[Full test code implementation]
```
