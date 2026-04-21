---
name: se-solid-principle-check
description: Analyse and validate code against SOLID principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion). Use this skill whenever the software engineer agent is reviewing, refactoring, or implementing code and needs to verify or enforce SOLID compliance. Trigger on any code review, class/module design, or refactoring task.
---

# SOLID Principle Check

Evaluate code against each SOLID principle and surface violations with actionable fixes.

## Process

### 1. Single Responsibility Principle (SRP)
- Each class/module should have **one reason to change**
- Red flags: classes doing I/O + business logic + formatting; methods >20 lines mixing concerns
- Fix: extract into focused classes/services

### 2. Open/Closed Principle (OCP)
- Open for **extension**, closed for **modification**
- Red flags: switch/if-else chains that grow with new types; modifying existing classes for new features
- Fix: strategy pattern, polymorphism, or plugin architecture

### 3. Liskov Substitution Principle (LSP)
- Subtypes must be **substitutable** for their base types
- Red flags: overridden methods that throw `NotImplemented`; subclass that weakens preconditions or strengthens postconditions
- Fix: redesign inheritance, prefer composition

### 4. Interface Segregation Principle (ISP)
- Clients should not depend on interfaces they **don't use**
- Red flags: fat interfaces with unrelated methods; classes implementing methods as no-ops
- Fix: split into focused interfaces/protocols

### 5. Dependency Inversion Principle (DIP)
- Depend on **abstractions**, not concretions
- Red flags: `new ConcreteClass()` inside business logic; hard-coded external dependencies
- Fix: inject dependencies via constructor or DI container

## Output Format

For each violation found:
```
[PRINCIPLE] [FILE:LINE] — Description of violation
→ Suggested fix
```

Summarise at the end:
- ✅ Principles followed
- ⚠️ Violations found (with count)
- Refactor priority: High / Medium / Low
