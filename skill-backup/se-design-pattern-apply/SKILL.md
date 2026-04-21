---
name: se-design-pattern-apply
description: Identify the right design pattern (creational, structural, behavioral) for a given problem and implement it correctly within the project's codebase. Use this skill whenever the software engineer agent is designing a new module, solving a recurring structural problem, or when the user asks how to structure code. Trigger on any feature implementation that involves object creation, composition, or behavior coordination.
---

# Design Pattern Apply

Select and implement the most appropriate design pattern for the problem at hand.

## Pattern Selection Guide

### Creational — *How objects are created*
| Pattern | Use When |
|---|---|
| Factory Method | Object type determined at runtime |
| Abstract Factory | Families of related objects |
| Builder | Complex object with many optional parts |
| Singleton | Single shared instance (use sparingly) |
| Prototype | Clone existing objects cheaply |

### Structural — *How objects are composed*
| Pattern | Use When |
|---|---|
| Adapter | Incompatible interfaces need to work together |
| Decorator | Add behaviour without subclassing |
| Facade | Simplify a complex subsystem |
| Proxy | Control access, lazy-load, or cache |
| Composite | Tree structures of uniform elements |
| Bridge | Decouple abstraction from implementation |

### Behavioral — *How objects communicate*
| Pattern | Use When |
|---|---|
| Strategy | Swap algorithms at runtime |
| Observer | One-to-many event notification |
| Command | Encapsulate requests as objects (undo/redo) |
| Chain of Responsibility | Pass request through a handler pipeline |
| Template Method | Define skeleton, let subclasses fill steps |
| State | Behaviour changes based on internal state |
| Repository | Abstract data access layer |
| Mediator | Reduce coupling between many objects |

## Implementation Process

1. **Identify the problem type** — creation, structure, or behaviour
2. **Match to pattern** using the table above
3. **Check project pattern** — read existing code to match naming and style conventions
4. **Implement** — provide full working code, not pseudocode
5. **Show before/after** — demonstrate how the pattern solves the original problem

## Output Format

```
Pattern chosen: [NAME]
Reason: [one sentence]

[implementation code]

Before (problem):
[old code snippet]

After (pattern applied):
[new code snippet]
```
