# Refactor Plan

Refactor safely and incrementally — never break working code.

## Refactor Triggers

Refactor when you see:
- Long methods (>20 lines doing multiple things)
- Deep nesting (>3 levels)
- Duplicated logic (DRY violation)
- God classes (one class doing everything)
- Primitive obsession (raw strings/ints instead of value objects)
- Shotgun surgery (one change requires edits in many files)
- Feature envy (method uses another class's data more than its own)

## Process

### Step 1 — Understand before touching
- Read and understand the current code fully
- Identify what tests exist (if any)
- Map all callers/consumers of the code being changed

### Step 2 — Define the goal
- What smell are you removing?
- What is the desired end state?
- What must NOT change (public interface, behaviour, output)?

### Step 3 — Plan incremental steps
Break the refactor into small, independently verifiable steps:
```
Step 1: Extract method X from method Y
Step 2: Move class A to module B
Step 3: Replace primitive with value object
...
```
Each step should leave the code in a working state.

### Step 4 — Execute
- One step at a time
- Run tests (or verify manually) after each step
- Commit after each step if possible

### Step 5 — Verify
- Behaviour is unchanged
- Tests pass
- Code is cleaner and easier to understand

## Common Refactoring Recipes

| Smell | Technique |
|---|---|
| Long method | Extract Method |
| Large class | Extract Class |
| Duplicate code | Extract Method / Pull Up Method |
| Conditional complexity | Replace Conditional with Polymorphism |
| Primitive obsession | Introduce Value Object |
| Data clumps | Introduce Parameter Object |
| Temporary field | Extract Class |

## Output Format

```
## Refactor Goal
[What smell / what end state]

## Risk Assessment
Low / Medium / High — [reason]

## Steps
1. [Step description] — [files affected]
2. ...

## Code Changes
[before → after for each step]
```
