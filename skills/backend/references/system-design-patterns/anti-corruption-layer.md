# Anti-Corruption Layer

## Anti-Corruption Layer Standards

- The Anti-Corruption Layer (ACL) must be used whenever integrating with an external or legacy system
- The ACL must prevent external domain concepts from leaking into the clean domain
- The ACL must translate between the external model and the internal model in both directions
- The ACL must be implemented at the appropriate level: internal library, proxy, or sidecar

---

## Instructions

### Step 1 - Identification

> Understand whether an ACL is needed

- Identify whether the system integrates with external or legacy systems
- Identify whether the external system has a different domain model
- Identify whether the external system's model is unstable or poorly designed

---

### Step 2 - Evaluate

> Determine if an ACL is the right fit

- Best for: Integrating with third-party APIs, legacy systems, or any external boundary
- Best for: Protecting a clean domain model from external corruption
- NOT for: Internal communication between services that share the same domain model
- NOT for: Simple pass-through where no model translation is needed

---

### Step 3 - Choose ACL Implementation Level

> Determine where the ACL should live based on operational context

#### Internal Library (Application Side)

- The translation logic lives inside the application as mapper/translator classes
- Right call when:
  - One or few external systems to integrate with
  - The mapping is not complex enough to justify a separate deployment unit
  - Your team already has clean separation via dedicated classes inside the app
  - You want to keep operational complexity low (no extra networking hops, containers, or observability overhead)

#### Proxy

- The ACL sits as a standalone proxy service between your application and the external system
- Right call when:
  - Multiple internal services need to reach the same external system
  - You want to centralize rate limiting, caching, and protocol translation
  - You need to enforce policies (auth transformation, request shaping) in one place

#### Sidecar (Service Mesh Pattern)

- The ACL runs as a sidecar container alongside each service instance
- Right call when:
  - Multiple services need the same translation logic for the same external system (e.g., a legacy ERP, a third-party API) — each service gets its own sidecar instance running the same translation logic, avoiding duplication
  - The external system uses a different protocol (SOAP, gRPC, binary format) and you don't want domain services to care about that — the sidecar handles protocol + model translation transparently
  - Your team is strict about zero pollution in domain code — not even a mapper class — the sidecar enforces this at the deployment level
  - The external contract changes frequently — you only update and redeploy the sidecar independently without touching the domain service

---

### Step 4 - Output

> Provide the ACL design

- Define the boundary where the ACL sits
- Define the implementation level (internal library, proxy, or sidecar)
- Define the external model (as-is)
- Define the internal model (clean)
- Define the translation logic in both directions

---

## Edge Cases

### 1. Performance Overhead of Translation

> Adding a translation layer between every external call

**Rule of Thumb:**

- The translation cost is always cheaper than domain corruption

**Practical Trade-off:**

- Extra mapping code and potential latency vs clean domain that is easy to maintain and test

---

### 2. Choosing Between Internal Library and Sidecar

> Over-engineering with a sidecar when an internal library is sufficient

**Rule of Thumb:**

- Default to internal library unless one of the sidecar triggers is clearly met
- Sidecar adds networking hops, more containers to manage, and observability overhead

**Practical Trade-off:**

- Internal library: simpler operations, but external model awareness exists in code
- Sidecar: zero domain pollution, but higher operational complexity

---

### 3. Multiple Services Sharing the Same External System

> Duplicating ACL logic across every service that talks to the same external system

**Rule of Thumb:**

- When 3+ services need the same translation, extract to a shared proxy or sidecar
- When 1-2 services need it, internal library is acceptable

**Practical Trade-off:**

- Shared proxy: single point of failure but single source of truth for translation
- Sidecar: no single point of failure but more containers to manage

---

## Bad Example

### 1. Leaking External Model Into Domain

```typescript
class User {
  usr_id: string;       // external naming convention
  is_active_flag: number; // external boolean representation
}
```

### 2. Sidecar for a Single Service With a Simple External Integration

> Deploying a sidecar container alongside a single service that talks to one simple REST API — the operational overhead outweighs the benefit. Use an internal library instead.

---

## Good Example

### 1. ACL as Internal Library (Most Common)

```typescript
interface LegacyUserResponse {
  usr_id: string;
  usr_fullname: string;
  is_active_flag: number;
}

class User {
  constructor(
    readonly id: string,
    readonly fullName: string,
    readonly isActive: boolean
  ) {}
}

class UserACL {
  toDomain(external: LegacyUserResponse): User {
    return new User(
      external.usr_id,
      external.usr_fullname,
      external.is_active_flag === 1
    );
  }

  toExternal(user: User): LegacyUserResponse {
    return {
      usr_id: user.id,
      usr_fullname: user.fullName,
      is_active_flag: user.isActive ? 1 : 0,
    };
  }
}
```

### 2. ACL as Proxy (Centralized for Multiple Services)

```markdown
Service A ──┐
            ├──→ [ACL Proxy Service] ──→ Legacy ERP
Service B ──┤     - protocol translation (REST → SOAP)
            │     - model mapping
Service C ──┘     - rate limiting
                  - centralized error handling
```

### 3. ACL as Sidecar (Service Mesh Pattern)

```markdown
┌─────────────────────────────┐
│ Pod                         │
│                             │
│  ┌──────────┐  ┌─────────┐  │
│  │ Domain   │  │ Sidecar │  │
│  │ Service  │──│  ACL    │───→ External System
│  │          │  │         │  │
│  └──────────┘  └─────────┘  │
│                             │
│ Domain calls localhost only │
│ Sidecar handles translation │
└─────────────────────────────┘

Benefits:
- Domain service has zero knowledge of external model
- Sidecar can be updated independently
- Same sidecar image reused across all services that need this ACL
```
