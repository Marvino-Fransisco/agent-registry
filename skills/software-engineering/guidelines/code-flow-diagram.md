# Code Flow Diagram

Visualise how code executes — before implementing or to explain existing code.

## When to Use Each Diagram Type

| Type | Use For |
|---|---|
| Flowchart | Control flow, conditionals, loops |
| Sequence diagram | Inter-module/service communication |
| State diagram | Object/entity lifecycle |
| Data flow | How data transforms through layers |

## Diagram Format (Mermaid)

### Flowchart (control flow)
```mermaid
flowchart TD
    A[Request received] --> B{Validate input}
    B -- Valid --> C[Call service]
    B -- Invalid --> D[Return 400]
    C --> E{User exists?}
    E -- Yes --> F[Process]
    E -- No --> G[Throw UserNotFoundError]
    F --> H[Return 200]
    G --> I[Return 404]
```

### Sequence Diagram (layer communication)
```mermaid
sequenceDiagram
    participant C as Controller
    participant S as Service
    participant R as Repository
    participant DB as Database

    C->>S: createUser(dto)
    S->>R: findByEmail(email)
    R->>DB: SELECT * WHERE email=?
    DB-->>R: null
    R-->>S: null
    S->>R: save(user)
    R->>DB: INSERT INTO users
    DB-->>R: user row
    R-->>S: User entity
    S-->>C: UserDTO
```

### State Diagram (entity lifecycle)
```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Submitted: submit()
    Submitted --> Approved: approve()
    Submitted --> Rejected: reject()
    Approved --> Cancelled: cancel()
    Rejected --> [*]
    Cancelled --> [*]
```

## Process

1. **Identify the scope** — one feature? one function? one layer?
2. **Choose diagram type** based on what needs to be shown
3. **Map the happy path** first
4. **Add edge cases and error paths**
5. **Annotate** key decision points and data shapes

## Output Format

Provide:
1. The diagram in Mermaid syntax (fenced with ` ```mermaid `)
2. A brief written description of the flow (2-5 sentences)
3. Key decision points called out explicitly

```
## Flow: [feature name]

### Diagram
[mermaid code]

### Description
[2-5 sentence walkthrough]

### Key Decision Points
- [condition] → [outcome]
- [condition] → [outcome]
```
