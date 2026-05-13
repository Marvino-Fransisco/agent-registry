# Pattern Refactoring Recognition

## Pattern Refactoring Recognition Standards

- Refactoring recommendations must be based on actual pain points — not aesthetic preferences
- Refactoring must never be recommended without understanding the current context
- Every refactoring suggestion must include the risk and effort assessment

---

## Instructions

### Step 1 - Identification

> Identify whether refactoring is actually needed

- Identify whether the current patterns cause real problems (bugs, slow development, test difficulty)
- Identify whether the refactoring request comes from the user or is self-identified
- Identify the scope of the refactoring (single file, module, entire project)

---

### Step 2 - Assess Refactoring Triggers

> Evaluate common triggers that indicate a pattern needs refactoring

- Code duplication across modules (same logic written differently in multiple places)
- Inconsistent error handling (different patterns for the same concern)
- Tight coupling (modules that cannot be tested or modified independently)
- God objects (files or classes that handle too many responsibilities)
- Missing abstractions (direct infrastructure access from business logic)
- Circular dependencies
- Untestable code (hardcoded dependencies, no DI, side effects in constructors)
- Inconsistent naming conventions

---

### Step 3 - Prioritize

> Rank refactoring candidates by impact and risk

- High impact, low risk: Do first (e.g., extract shared utility, standardize error format)
- High impact, high risk: Plan carefully (e.g., change architecture pattern, split module)
- Low impact, low risk: Do opportunistically (e.g., rename files for consistency)
- Low impact, high risk: Defer or skip (e.g., rewrite working legacy code for style)

---

### Step 4 - Output

> Provide the refactoring recommendation

- List each refactoring candidate with:
  - Current state (what exists now)
  - Target state (what it should become)
  - Trigger (why it needs to change)
  - Impact (what improves)
  - Risk (what could break)
  - Effort (small, medium, large)
- Provide the recommended execution order

---

## Edge Cases

### 1. Working Code That Should Not Be Refactored

> Code that looks "wrong" but is intentionally written that way for a specific reason

**Example:**

- Denormalized table for read performance
- Duplicated validation in controller and service for defense-in-depth

**Rule of Thumb:**

- Always check if there is a reason for the current pattern before recommending change
- Ask the team before refactoring legacy code

**Practical Trade-off:**

- Cleaner code vs breaking working functionality
- If it works and is not actively modified, leave it alone

---

### 2. Incremental vs Big Bang Refactoring

> Choosing between gradual migration or full rewrite

**Example:**

- Migrating from layered to Hexagonal architecture module by module vs all at once

**Rule of Thumb:**

- Always prefer incremental refactoring
- Big bang is only acceptable for small, isolated modules

**Practical Trade-off:**

- Incremental: Longer migration period, but system stays functional throughout
- Big bang: Faster completion, but higher risk of breaking everything

---

## Bad Example

### 1. Refactoring Without Understanding Context

> Recommending to replace raw SQL with an ORM without understanding query performance requirements

### 2. Big Bang Refactoring for a Large Codebase

> "Let's rewrite the entire application using Clean Architecture over the weekend"

---

## Good Example

### 1. Prioritized Refactoring Plan

| Candidate | Current | Target | Trigger | Impact | Risk | Effort |
|-----------|---------|--------|---------|--------|------|--------|
| Error handling | Mixed try-catch and no handling | Centralized AppError + middleware | Inconsistent error responses | High | Low | Small |
| User module coupling | Service imports DB directly | Repository pattern | Cannot unit test service | High | Medium | Medium |
| File naming | Mixed camelCase and kebab-case | Consistent kebab-case | Developer confusion | Low | Low | Small |

Execution order: Error handling → File naming → User module coupling
