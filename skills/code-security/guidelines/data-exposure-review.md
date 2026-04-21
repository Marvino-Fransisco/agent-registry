# Sensitive Data Exposure Review

## What Counts as Sensitive Data

| Category | Examples |
|----------|---------|
| **PII** | Full name + email + address combo, SSN, passport, date of birth |
| **Authentication** | Passwords (any form), tokens, session IDs, API keys, secrets |
| **Financial** | Full card numbers (PAN), CVV, bank account numbers, transaction amounts for private users |
| **Health** | Medical records, diagnoses, prescriptions, therapy notes |
| **Business-sensitive** | Internal pricing, margins, unreleased product data, employee salaries |
| **Infrastructure** | Server IPs, internal hostnames, stack traces, DB schema details |
| **Behavioral** | Precise location history, private messages, browsing history |

---

## Layer 1 — API Response Over-Exposure

The most common data exposure: returning more than the client needs.

### Audit Pattern

For every API endpoint, ask: **does the response include any field the client does not display or need?**

```js
// ❌ Full ORM object returned — includes passwordHash, internal flags, admin notes
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);  // Everything in the DB row goes to the client
});

// ✅ Explicit DTO projection
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id, {
    select: ['id', 'name', 'email', 'avatarUrl', 'createdAt'],
  });
  res.json(user);
});
```

### Common Over-Exposed Fields to Check

- `passwordHash` / `password` — should never appear in any response
- `refreshToken` — should never be in response body (use `HttpOnly` cookie)
- `internalNotes` / `adminComments`
- `isAdmin`, `isBanned`, `internalFlags` — exposing to non-admins
- `stripeCustomerId`, `stripePaymentMethodId` — internal IDs that expose your payment provider
- `resetToken`, `verificationToken` — one-time tokens that should be consumed, not persisted in responses
- `createdByIp`, `lastLoginIp` — IP addresses are PII in many jurisdictions

---

## Layer 2 — Error Message Leakage

Errors are a major source of unintentional data exposure.

### What Errors Can Leak

| Error Type | Risk |
|-----------|------|
| Stack trace | File paths, function names, library versions, internal architecture |
| DB error | Table names, column names, query structure, DB version |
| Auth error | Whether a user exists (account enumeration) |
| Validation error | Internal field names, schema structure |
| 500 with raw exception | Secrets in env if included in message |

```js
// ❌ Exposes stack trace and internal details
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack,
    code: err.code,
  });
});

// ✅ Generic client message, full detail in server log
app.use((err, req, res, next) => {
  const errorId = crypto.randomUUID();
  logger.error('Unhandled error', {
    errorId,
    message: err.message,
    stack: err.stack,
    code: err.code,
    path: req.path,
    userId: req.user?.id,
  });
  res.status(err.statusCode ?? 500).json({
    error: 'An unexpected error occurred',
    errorId,    // So support can correlate without exposing detail to client
  });
});
```

**Checklist**:
- [ ] No stack traces in API responses (any environment)
- [ ] No database error messages in API responses
- [ ] No internal field/table names in validation errors
- [ ] Error responses use generic messages — specific to error type only (400, 401, 403, 404, 500)
- [ ] Error IDs (UUID) in responses for support correlation — not error details
- [ ] Server header removed or genericized (don't advertise "nginx/1.24.0" or "Express 4.18")

---

## Layer 3 — Logging Sensitive Data

Logs are often the most overlooked data exposure vector. They persist, are shipped to third-party services (Datadog, Splunk, Logtail), and may be accessed by support staff.

### What Must Never Be Logged

```js
// ❌ Logging sensitive data
logger.info('User login', {
  email: req.body.email,
  password: req.body.password,   // NEVER
  token: req.headers.authorization,  // NEVER
  creditCard: req.body.card,     // NEVER
});

// ✅ Log only what you need
logger.info('User login', {
  userId: user.id,
  email: maskEmail(user.email),  // user@domain.com → u***@domain.com
  ip: req.ip,
  success: true,
});
```

### Masking Patterns

```js
// Email masking
const maskEmail = (email) => {
  const [user, domain] = email.split('@');
  return `${user[0]}***@${domain}`;
};

// Card masking (last 4 only)
const maskCard = (pan) => `****-****-****-${pan.slice(-4)}`;

// Token masking (first 6 chars only)
const maskToken = (token) => `${token.slice(0, 6)}...`;
```

**Log audit checklist**:
- [ ] No passwords in logs (even hashed passwords)
- [ ] No full tokens or API keys in logs (mask or omit)
- [ ] No full credit card numbers in logs
- [ ] No SSN, full DOB, or passport numbers in logs
- [ ] Email addresses masked or excluded from high-volume log lines
- [ ] Request body not logged wholesale (`JSON.stringify(req.body)` with no filter)
- [ ] Log aggregation service (Datadog, etc.) has appropriate access controls

---

## Layer 4 — Frontend Bundle Exposure

Frontend JavaScript bundles are public. Anything in your frontend code is visible to any user.

### What to Audit in Frontend Code

```js
// ❌ Hardcoded in frontend
const INTERNAL_ADMIN_ENDPOINT = 'https://api.example.com/internal/admin';
const SECRET_KEY = 'sk_live_abc123';   // Visible to anyone
const FEATURE_FLAGS = {
  unreleased_feature: true,  // Leaks product roadmap
};

// ❌ Sending more data than displayed
// API returns full user object including salary, SSN, etc.
// Frontend only displays name and email
// All the extra fields are still in browser memory / network tab
```

**Checklist**:
- [ ] No API keys, secrets, or credentials in frontend bundles
- [ ] Internal API endpoint paths not exposed unless necessary
- [ ] Feature flags for unreleased features not visible in frontend code
- [ ] Source maps not deployed to production (expose original source)
- [ ] API responses scoped to what frontend actually displays

### Source Map Policy

```js
// webpack.config.js / next.config.js
module.exports = {
  productionSourceMap: false, // Don't expose source maps in production
  // If needed for error tracking, upload to Sentry privately — don't serve publicly
};
```

---

## Layer 5 — Database Query Exposure

Over-selecting from the database exposes data downstream.

```js
// ❌ SELECT * — all columns pulled, some will leak
const users = await db.query('SELECT * FROM users WHERE active = true');

// ✅ Explicit column selection
const users = await db.query(`
  SELECT id, name, email, created_at
  FROM users
  WHERE active = true
`);
```

**Checklist**:
- [ ] No `SELECT *` in production queries (explicit columns only)
- [ ] Sensitive columns (`password_hash`, `ssn`, `credit_card`) only selected in queries that genuinely need them
- [ ] Audit log of who accessed sensitive tables (DB-level if possible)

---

## Layer 6 — PII Handling Compliance

For products handling personal data, apply these principles:

| Principle | Implementation |
|-----------|---------------|
| **Data minimization** | Collect only what you need |
| **Purpose limitation** | Use data only for the stated purpose |
| **Storage limitation** | Delete data when no longer needed |
| **Accuracy** | Allow users to update their data |
| **Right to erasure** | Implement user data deletion |

**Checklist**:
- [ ] Data retention policy defined and implemented (automated deletion)
- [ ] User data export endpoint exists (GDPR Article 20)
- [ ] User data deletion implemented (GDPR Article 17 — right to erasure)
- [ ] Third-party services receiving user data have DPAs (Data Processing Agreements)
- [ ] Analytics services receive anonymized / pseudonymized data where possible

---

## Output Format for Design Doc

```markdown
## Data Exposure Controls

### API Response Audit

| Endpoint | Fields Returned | Sensitive Fields Excluded |
|----------|----------------|--------------------------|
| GET /api/users/:id | id, name, email, avatarUrl | passwordHash, internalFlags, adminNotes |
| GET /api/orders/:id | id, total, status, items | paymentMethodId, internalCost |

### Logging Policy

- **Logged**: userId, action, resource ID, timestamp, IP, outcome
- **Never logged**: passwords, tokens, card numbers, SSN
- **Masked**: email (u***@domain.com), card (****1234)

### PII Retention

| Data Type | Retention | Deletion Method |
|-----------|-----------|----------------|
| User profile | Account lifetime + 30 days | Soft delete → hard delete |
| Payment records | 7 years (legal requirement) | Anonymize non-required fields |
| Access logs | 90 days | Automated purge |
```
