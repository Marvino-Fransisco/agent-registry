---
name: se-code-review
description: Perform a thorough code review covering correctness, readability, performance, security, and maintainability. Use this skill whenever the software engineer agent is asked to review, audit, or give feedback on existing code. Trigger on any "review my code", "check this", "is this correct", or "what's wrong with" request.
---

# Code Review

Conduct a structured review across five dimensions.

## Review Dimensions

### 1. Correctness
- Does the code do what it's supposed to do?
- Are all edge cases handled (nulls, empty inputs, boundaries, concurrency)?
- Are there off-by-one errors, race conditions, or logic bugs?

### 2. Readability
- Are names descriptive and consistent with project conventions?
- Is the code self-documenting or does it need comments?
- Are functions short and focused?
- Is there unnecessary complexity (over-engineering)?

### 3. Performance
- Are there N+1 query patterns, unnecessary loops, or redundant computations?
- Is memory allocation excessive?
- Are expensive operations cached or deferred where appropriate?

### 4. Security
- Is user input validated and sanitised?
- Are there injection vulnerabilities (SQL, command, path traversal)?
- Are secrets/credentials hardcoded?
- Are permissions/access checks in place?

### 5. Maintainability
- Is the code testable (low coupling, injectable dependencies)?
- Does it follow the project's existing patterns?
- Will a new developer understand this in 6 months?
- Are there magic numbers or unexplained constants?

## Severity Levels

- 🔴 **Critical** — Bug, security hole, or data integrity risk. Must fix.
- 🟠 **Major** — Significant performance or maintainability issue. Should fix.
- 🟡 **Minor** — Style, naming, or small improvement. Nice to fix.
- 🟢 **Positive** — Good patterns worth noting.

## Output Format

```
## Summary
[2-3 sentence overall assessment]

## Issues

🔴 [FILE:LINE] — [Issue description]
→ [Suggested fix]

🟠 [FILE:LINE] — [Issue description]
→ [Suggested fix]

🟡 [FILE:LINE] — [Issue description]
→ [Suggested fix]

🟢 [FILE:LINE] — [What's done well]

## Verdict
Approve / Request Changes / Needs Major Rework
```
