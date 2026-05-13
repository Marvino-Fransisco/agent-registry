---
name: backend
description: >
  Use this skill when working on tasks, answering questions, or brainstorming related to backend systems and engineering. Focus on architecture, project setup, system design patterns, and code quality standards.
license: MIT
compatibility: Designed for AI Agent
metadata:
  author: Marvino Fransisco
  version: "1.0"
---

# Backend Skill

This skill contains routes to references that should be read **before** dealing
with backend-related problems.

## Standards

- Testable code patterns
- SOLID principle
- Race condition prevention code

## Tree of Thought

> Determine which condition applies before reading references.

### Condition 1 — Empty Project

> Triggered when starting a new backend project from scratch.

1. Read [project-structure](references/fundamental/project-structure.md) reference
2. Read [ddd-vs-simple-model](references/fundamental/ddd-vs-simple-model.md) reference
3. Read [coding-patterns](references/fundamental/coding-patterns.md) reference
4. Read [bootstrap](references/fundamental/bootstrap.md) reference
5. Determine what system design pattern could be useful → read relevant system design references below

### Condition 2 — Existing Project

> Triggered when working with an existing backend codebase.

1. Read [project-pattern-recognition](references/fundamental/project-pattern-recognition.md) reference
2. Read [code-pattern-recognition](references/fundamental/code-pattern-recognition.md) reference
3. Read [pattern-refactoring-recognition](references/fundamental/pattern-refactoring-recognition.md) reference
4. Read [existing-documentation-context](references/fundamental/existing-documentation-context.md) reference
5. Determine what system design pattern might be needed or not needed → read relevant system design references below

## Routes

### Templates
| Template | Description |
| -------- | ----------- |
| [markdown](assets/markdown-template.md) | Template for writing markdown documentation |

### Fundamental

| Reference | Description |
| --------- | ----------- |
| [project-structure](references/fundamental/project-structure.md) | read this reference when deciding project architecture (CSR, CQRS, Clean Architecture, Hexagonal Pattern) |
| [ddd-vs-simple-model](references/fundamental/ddd-vs-simple-model.md) | read this reference when deciding between DDD or a simple model approach |
| [coding-patterns](references/fundamental/coding-patterns.md) | read this reference when defining coding conventions (one file one function vs one file multiple functions) |
| [bootstrap](references/fundamental/bootstrap.md) | read this reference when bootstrapping files and folders (logging, routing, controllers, configs, env loader, middlewares) |
| [project-pattern-recognition](references/fundamental/project-pattern-recognition.md) | read this reference when recognizing patterns in an existing project structure |
| [code-pattern-recognition](references/fundamental/code-pattern-recognition.md) | read this reference when recognizing code-level patterns in an existing codebase |
| [pattern-refactoring-recognition](references/fundamental/pattern-refactoring-recognition.md) | read this reference when identifying refactoring opportunities in project and code patterns |
| [existing-documentation-context](references/fundamental/existing-documentation-context.md) | read this reference when gathering context from existing documentation |

### System Design Patterns

| Reference | Description |
| --------- | ----------- |
| [ambassador](references/system-design-patterns/ambassador.md) | read this reference when evaluating the Ambassador pattern |
| [anti-corruption-layer](references/system-design-patterns/anti-corruption-layer.md) | read this reference when evaluating the Anti-Corruption Layer pattern |
| [asynchronous-request-reply](references/system-design-patterns/asynchronous-request-reply.md) | read this reference when evaluating the Asynchronous Request Reply pattern |
| [backend-for-frontends](references/system-design-patterns/backend-for-frontends.md) | read this reference when evaluating the Backend for Frontends pattern |
| [bulkhead](references/system-design-patterns/bulkhead.md) | read this reference when evaluating the Bulkhead pattern |
| [cache-aside](references/system-design-patterns/cache-aside.md) | read this reference when evaluating the Cache-Aside pattern |
| [choreography](references/system-design-patterns/choreography.md) | read this reference when evaluating the Choreography pattern |
| [circuit-breaker](references/system-design-patterns/circuit-breaker.md) | read this reference when evaluating the Circuit Breaker pattern |
| [claim-check](references/system-design-patterns/claim-check.md) | read this reference when evaluating the Claim Check pattern |
| [compensating-transaction](references/system-design-patterns/compensating-transaction.md) | read this reference when evaluating the Compensating Transaction pattern |
| [competing-consumers](references/system-design-patterns/competing-consumers.md) | read this reference when evaluating the Competing Consumers pattern |
| [compute-resource-consolidation](references/system-design-patterns/compute-resource-consolidation.md) | read this reference when evaluating the Compute Resource Consolidation pattern |
| [cqrs](references/system-design-patterns/cqrs.md) | read this reference when evaluating the CQRS pattern |
| [retry](references/system-design-patterns/retry.md) | read this reference when evaluating the Retry pattern |
| [saga](references/system-design-patterns/saga.md) | read this reference when evaluating the Saga pattern |
