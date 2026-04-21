---
name: se-dependency-analysis
description: Analyse code dependencies to detect coupling issues, circular dependencies, inappropriate imports, and violation of architectural boundaries. Use this skill whenever the software engineer agent is reviewing module structure, setting up a new feature, or diagnosing why a change caused unexpected breakage. Trigger on any "dependency", "coupling", "circular import", "architecture boundary", or "module structure" concern.
---

# Dependency Analysis

Map and evaluate dependencies to keep the codebase loosely coupled and well-structured.

## What to Analyse

### 1. Coupling Level
- **Tight coupling** — class A directly instantiates class B (bad)
- **Loose coupling** — class A depends on interface B, receives B via injection (good)
- Check: are concrete types leaking across layer boundaries?

### 2. Circular Dependencies
- Module A imports B which imports A — causes init order bugs and makes testing hard
- Detection: trace import chains, look for cycles
- Fix: extract shared code to a third module, or invert the dependency

### 3. Architectural Boundary Violations
Common layered architecture (must flow one way):
```
Controller → Service → Repository → Database
                ↓
           External APIs
```
Violations:
- Repository calling a Service
- Service importing a Controller
- Domain model importing infrastructure code

### 4. Dependency Direction
- High-level modules (business logic) must NOT depend on low-level modules (DB drivers, HTTP clients)
- Both should depend on abstractions (interfaces/protocols)

### 5. Fan-in / Fan-out
- **High fan-in** (many things depend on one module) → that module is critical, changes are risky
- **High fan-out** (one module depends on many things) → fragile, hard to test
- Ideal: modules have moderate, intentional dependencies

## Analysis Process

1. List all imports/dependencies of the module under review
2. Categorise each: same layer / cross-layer / external
3. Check for cycles
4. Check direction (does it violate the architecture?)
5. Flag tight coupling (concrete types instead of abstractions)

## Output Format

```
## Dependency Map for [module]

Imports:
- [module] → [type: same-layer / cross-layer / external]

## Issues Found

🔴 Circular dependency: A → B → A
→ Fix: extract [X] to shared module

🟠 Boundary violation: [Repository] imports [Service]
→ Fix: invert dependency, pass data via parameter

🟡 Tight coupling: new ConcreteEmailService() in OrderService
→ Fix: inject via constructor, depend on IEmailService

## Dependency Health
Coupling: Low / Medium / High
Cycles: None / [list]
Boundary violations: None / [list]
```
