---
name: se-pattern-read
description: Read and understand the existing code patterns, conventions, and architecture of a project before implementing anything. Use this skill at the START of every implementation task, before writing any code. Trigger whenever the software engineer agent is about to implement a feature in an unfamiliar or partially-familiar codebase. This skill must run before se-design-pattern-apply or any code generation.
---

# Pattern Read

Understand how the project is built before adding to it. Code that follows existing patterns is code that belongs.

## What to Read

### 1. Project Structure
```bash
# Get the overall tree (code logic only, ignore assets/build/vendor)
find . -type f -name "*.ts" -o -name "*.py" -o -name "*.go" \
  | grep -v node_modules | grep -v dist | grep -v .git \
  | head -80
```
Understand:
- How is the project layered? (controllers / services / repos / models)
- Where does each concern live?
- What is the naming convention for files and folders?

### 2. Naming Conventions
- Files: `camelCase.ts` / `snake_case.py` / `kebab-case.go`?
- Classes: `UserService`, `user_service`, `UserSvc`?
- Interfaces/Types: `IUserRepo`, `UserRepository`, `UserRepo`?
- Tests: `*.spec.ts`, `*_test.go`, `test_*.py`?

### 3. Module/Class Structure
Pick 2-3 existing feature modules similar to what you're building. Read:
- How is the class constructed? (constructor injection, factory, etc.)
- What base classes or mixins are used?
- How are errors thrown and caught?
- How is validation done?

### 4. Data Access Pattern
- Is there a repository pattern?
- Is the ORM used directly in services, or abstracted?
- Are raw queries ever used? When?

### 5. Error Handling Pattern
- Custom error classes or generic Error?
- Where are errors caught — service layer, controller, global handler?
- What is the error response shape?

### 6. Dependency Injection
- Is there a DI container (e.g., tsyringe, NestJS IoC, Spring)?
- Or manual injection via constructors?
- How are singletons managed?

## Output Format

After reading, summarise:

```
## Project Pattern Summary

### Layer Structure
[description of layers and what lives where]

### Naming Conventions
- Files: [pattern]
- Classes: [pattern]
- Tests: [pattern]

### Key Patterns Observed
- DI: [how it's done]
- Error handling: [how it's done]
- Data access: [how it's done]
- Validation: [how it's done]

### Files to Use as Reference
- [file path] — [why it's a good reference]
- [file path] — [why it's a good reference]
```

Use this summary to anchor all implementation decisions that follow.
