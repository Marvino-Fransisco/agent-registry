---
name: sec-auth-review
description: Review and design authentication and authorization controls. Use when auditing login flows, session management, JWT handling, OAuth/SSO, role-based access, resource ownership checks, or privilege escalation risks. Trigger for any mention of auth, login, session, JWT, OAuth, RBAC, permissions, IDOR, or access control. Covers authentication (identity verification) and authorization (permission enforcement) separately because they fail in different ways.
---

# Authentication & Authorization Review

## The Critical Distinction

- **Authentication (AuthN)** — *Who are you?* Verifying identity.
- **Authorization (AuthZ)** — *What can you do?* Enforcing permissions.

They fail independently. A system can authenticate perfectly and still expose every user's data through missing authZ checks. Treat them as separate layers.

---

## Authentication

### Password Handling

```js
// ❌ Never
const hash = crypto.createHash('md5').update(password).digest('hex');
const hash = sha256(password); // no salt = rainbow table vulnerable

// ✅ Always
import bcrypt from 'bcrypt';
const hash = await bcrypt.hash(password, 12); // cost factor ≥ 10
const match = await bcrypt.compare(password, storedHash);
```

**Acceptable algorithms**: `bcrypt` (cost ≥ 10), `argon2id` (memory ≥ 64MB), `scrypt`  
**Never use**: MD5, SHA-1, SHA-256 without salt, any reversible encryption

**Checklist**:
- [ ] Passwords hashed with bcrypt/argon2id/scrypt — never plaintext or reversible
- [ ] Password comparison uses constant-time function (`bcrypt.compare`, not `===`)
- [ ] Minimum password policy enforced server-side (length ≥ 8, not just client)
- [ ] Password reset tokens: cryptographically random, short-lived (≤ 15min), single-use
- [ ] Old password required for password change (not just for reset)

---

### Account Enumeration Prevention

Account enumeration lets attackers learn which emails/usernames exist.

```js
// ❌ Leaks whether email exists
if (!user) return res.status(404).json({ error: 'User not found' });
if (!bcrypt.compare(password, user.hash)) return res.status(401).json({ error: 'Wrong password' });

// ✅ Generic message for both cases
if (!user || !await bcrypt.compare(password, user.hash)) {
  return res.status(401).json({ error: 'Invalid credentials' });
}
```

**Checklist**:
- [ ] Login returns identical error for "user not found" and "wrong password"
- [ ] Forgot password always returns "If that email exists, you'll receive a link" — never confirms
- [ ] Response time consistent regardless of whether user exists (dummy hash comparison if no user)
- [ ] Registration does not confirm email already registered — redirect to login instead

---

### Session Management

```js
// ❌ Predictable session ID
const sessionId = `session_${userId}_${Date.now()}`;

// ✅ Cryptographically random
import crypto from 'crypto';
const sessionId = crypto.randomBytes(32).toString('hex');
```

**Cookie security attributes**:
```
Set-Cookie: session=<token>; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| Attribute | Purpose |
|-----------|---------|
| `HttpOnly` | Prevents JS access — blocks XSS session theft |
| `Secure` | HTTPS only |
| `SameSite=Strict` | Blocks CSRF for cross-site requests |
| `Max-Age` | Limits session lifetime |

**Checklist**:
- [ ] Session IDs are 256-bit+ cryptographically random
- [ ] Session invalidated server-side on logout (not just cookie deletion)
- [ ] Session ID rotated after privilege change (login, role change)
- [ ] Concurrent session limit considered for high-security features
- [ ] Idle timeout enforced server-side

---

### JWT Security

```js
// ❌ Algorithm confusion attack — never accept "none" or RS256 on HS256 service
jwt.verify(token, secret); // vulnerable if algorithm not pinned

// ✅ Pin the algorithm explicitly
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

**JWT checklist**:
- [ ] Algorithm explicitly pinned — never `algorithms: ['*']` or algorithm from token header
- [ ] `exp` claim present and enforced — access tokens ≤ 15 min
- [ ] `iss` and `aud` claims validated
- [ ] Refresh tokens stored server-side with rotation on use
- [ ] Refresh token revocation list for logout/compromise scenarios
- [ ] JWT secret is ≥ 256-bit random — not a dictionary word
- [ ] Sensitive data not stored in JWT payload (it is Base64, not encrypted)

---

### MFA Considerations

- [ ] TOTP codes are single-use — rejected after first verification
- [ ] TOTP time window: ±1 step (30s) at most
- [ ] Recovery codes are hashed at rest (not plaintext)
- [ ] MFA bypass via "remember this device" tokens are long-lived — audit their storage and revocation

---

### OAuth / OIDC

```js
// ❌ Missing state param — CSRF on OAuth flow
const authUrl = `${provider}/authorize?client_id=${id}&redirect_uri=${cb}`;

// ✅ state param prevents CSRF
const state = crypto.randomBytes(16).toString('hex');
req.session.oauthState = state;
const authUrl = `${provider}/authorize?client_id=${id}&redirect_uri=${cb}&state=${state}`;

// On callback:
if (req.query.state !== req.session.oauthState) throw new Error('CSRF detected');
```

**Checklist**:
- [ ] `state` parameter used and validated on callback
- [ ] `redirect_uri` strictly whitelisted — no open redirect
- [ ] `code` exchanged server-side — never in frontend JS
- [ ] Token response stored securely — not in `localStorage`
- [ ] PKCE used for public clients (SPAs, mobile)

---

## Authorization

### The Golden Rule

**Every** request to a resource must explicitly verify:
1. Is the requester authenticated?
2. Does the requester own or have permission to this specific resource?

Route-level auth middleware is not enough. It only checks #1.

### IDOR (Insecure Direct Object Reference)

```js
// ❌ Vulnerable — any authenticated user can access any order
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order);
});

// ✅ Ownership check at resource level
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findOne({
    where: { id: req.params.id, userId: req.user.id } // ownership enforced in query
  });
  if (!order) return res.status(404).json({ error: 'Not found' });
  res.json(order);
});
```

**Checklist**:
- [ ] Every resource query includes ownership constraint (`userId = req.user.id`)
- [ ] IDs in URLs/bodies are UUIDs or non-sequential (prevents enumeration)
- [ ] Bulk endpoints filter by ownership — not just paginate all records
- [ ] Admin endpoints behind separate `requireAdmin` middleware, not just `requireAuth`

---

### RBAC (Role-Based Access Control)

```js
// ❌ Role check in business logic scattered across codebase
if (req.user.role === 'admin') { ... }

// ✅ Centralized permission check
import { can } from '@/lib/permissions';

// permissions.js
export const PERMISSIONS = {
  'order:read:own': ['user', 'admin'],
  'order:read:any': ['admin'],
  'order:delete': ['admin'],
};

export function can(user, permission) {
  return PERMISSIONS[permission]?.includes(user.role) ?? false;
}

// Usage
if (!can(req.user, 'order:delete')) return res.status(403).json({ error: 'Forbidden' });
```

**Checklist**:
- [ ] Permissions defined centrally — not scattered as role string comparisons
- [ ] `403 Forbidden` returned for authZ failure (not `404` which leaks resource existence)
- [ ] Privilege escalation path audited — can a user grant themselves higher roles?
- [ ] Service accounts / API keys scoped to minimum needed permissions
- [ ] Permission checks tested with both correct and incorrect roles

---

## Authorization Audit Template

For each endpoint in the design, fill in:

```markdown
## Authorization Matrix

| Endpoint | Method | Anonymous | User (own) | User (other) | Admin |
|----------|--------|-----------|------------|-------------|-------|
| /api/orders | GET | 401 | 200 (own only) | 403 | 200 (all) |
| /api/orders/:id | GET | 401 | 200 | 404 | 200 |
| /api/orders/:id | DELETE | 401 | 403 | 403 | 200 |
| /api/users/:id | PATCH | 401 | 200 (own fields) | 403 | 200 |
```

This matrix is the ground truth for both implementation and test cases.
