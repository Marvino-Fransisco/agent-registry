# API Security Hardening

## Hardening Categories

1. Rate Limiting & Abuse Prevention
2. HTTP Method Enforcement
3. Request Validation
4. Response Hardening
5. GraphQL-Specific Controls
6. Versioning & Deprecation
7. Webhook Security

---

## 1. Rate Limiting

Rate limiting is a first-line defense against brute force, credential stuffing, scraping, and DDoS.

### Rate Limiting by Endpoint Type

| Endpoint Type | Suggested Limit | Window |
|---------------|----------------|--------|
| Login / Auth | 5 attempts | per IP, per 15 min |
| OTP / 2FA verification | 5 attempts | per account, per 10 min |
| Password reset request | 3 requests | per email, per hour |
| Account registration | 10 per IP | per hour |
| Public read endpoints | 100 req | per IP, per minute |
| Authenticated user endpoints | 300 req | per user, per minute |
| Payment / financial mutations | 10 req | per user, per minute |
| File upload | 20 req | per user, per hour |
| Admin endpoints | 60 req | per admin, per minute |

### Implementation Patterns

```js
// Express + express-rate-limit
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// Auth endpoint — strict
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({ ... }),        // Distributed — not in-memory
  keyGenerator: (req) => req.ip,         // Per IP
  message: { error: 'Too many attempts. Try again later.' },
  skipSuccessfulRequests: true,          // Only count failures
});

// General API — permissive
export const apiLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  keyGenerator: (req) => req.user?.id ?? req.ip, // Per user when authenticated
  store: new RedisStore({ ... }),
});
```

**Checklist**:
- [ ] Rate limiter uses Redis/distributed store — not in-memory (doesn't work with multiple servers)
- [ ] Rate limiting applied at API gateway / load balancer layer too (not just application)
- [ ] `Retry-After` header returned on 429 response
- [ ] IP-based limiting supplemented with account-based limiting for auth endpoints
- [ ] Rate limits monitored and alerted on threshold breaches

---

## 2. HTTP Method Enforcement

```js
// ❌ Any HTTP method works — DELETE accessible via GET
app.all('/api/orders/:id', handler);

// ✅ Explicit method — router enforces
app.get('/api/orders/:id', getOrder);
app.delete('/api/orders/:id', deleteOrder);

// Return 405 with Allow header for wrong method
app.use((req, res) => {
  res.setHeader('Allow', 'GET, POST');
  res.status(405).json({ error: 'Method Not Allowed' });
});
```

**Checklist**:
- [ ] Each endpoint handles only its intended HTTP methods
- [ ] `405 Method Not Allowed` with `Allow` header returned for wrong methods
- [ ] `HEAD` and `OPTIONS` handled safely (don't expose unexpected capabilities)
- [ ] TRACE method disabled at web server level

---

## 3. Request Validation

### Size Limits

```js
// ❌ No limit — attacker sends 1GB JSON body
app.use(express.json());

// ✅ Strict limits
app.use(express.json({ limit: '10kb' }));          // Global API limit
app.use('/api/upload', express.raw({ limit: '5mb' })); // Upload endpoint
```

```nginx
# Nginx — enforce at server level too
client_max_body_size 10M;
```

### Schema Validation

Every endpoint should validate its input before processing. Use a schema library.

```js
import { z } from 'zod';

const CreateOrderSchema = z.object({
  productId: z.string().uuid(),
  quantity: z.number().int().positive().max(100),
  shippingAddress: z.object({
    line1: z.string().min(1).max(200),
    city: z.string().min(1).max(100),
    country: z.string().length(2), // ISO 3166-1 alpha-2
  }),
});

app.post('/api/orders', requireAuth, async (req, res) => {
  const result = CreateOrderSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: 'Invalid request', details: result.error.flatten() });
  }
  const data = result.data; // Type-safe, validated
  // ...
});
```

**Checklist**:
- [ ] All endpoints have explicit input schema (not just `req.body.field` access)
- [ ] `400 Bad Request` with clear (but non-verbose) error on validation failure
- [ ] String fields have `max` length — never unbounded
- [ ] Numeric fields have `min`/`max` bounds — no negative IDs or absurd quantities
- [ ] UUIDs validated as UUIDs before DB query
- [ ] Query parameters sanitized — don't trust `req.query.page` as a number without parsing

---

## 4. Response Hardening

### Information Leakage in Responses

```js
// ❌ Expose internal details
catch (err) {
  res.status(500).json({ error: err.message, stack: err.stack });
}

// ✅ Generic client message, detailed server log
catch (err) {
  logger.error('Order creation failed', { err, userId: req.user.id, body: req.body });
  res.status(500).json({ error: 'Something went wrong. Please try again.' });
}
```

### Data Minimization

```js
// ❌ Return full user object
res.json(await User.findById(id));
// Includes: passwordHash, internalFlags, adminNotes, etc.

// ✅ Explicit DTO — only fields the client needs
const user = await User.findById(id);
res.json({
  id: user.id,
  name: user.name,
  email: user.email,
  createdAt: user.createdAt,
});
```

**Checklist**:
- [ ] No stack traces in API responses
- [ ] No internal IDs, file paths, or server details in error messages
- [ ] Database error messages not surfaced (never expose table/column names)
- [ ] API response contains only fields the client needs
- [ ] `X-Powered-By` header removed (`app.disable('x-powered-by')`)
- [ ] `Server` header removed or genericized at web server level

---

## 5. GraphQL-Specific Controls

GraphQL has unique attack surfaces not present in REST.

```js
// ❌ Deeply nested query — O(n^k) database queries
query {
  user {
    friends {
      friends {
        friends {
          friends { id name email }
        }
      }
    }
  }
}
```

**Checklist**:
- [ ] Query depth limit enforced (`graphql-depth-limit` — max depth: 5–7)
- [ ] Query complexity limit enforced (`graphql-query-complexity`)
- [ ] Introspection disabled in production
- [ ] Batching abuse limited (max operations per batch request)
- [ ] Field-level authorization — not just resolver-level
- [ ] N+1 queries prevented with DataLoader
- [ ] Mutations rate limited separately from queries

```js
import depthLimit from 'graphql-depth-limit';
import { createComplexityLimitRule } from 'graphql-validation-complexity';

const server = new ApolloServer({
  validationRules: [
    depthLimit(7),
    createComplexityLimitRule(1000),
  ],
  introspection: process.env.NODE_ENV !== 'production',
});
```

---

## 6. API Versioning Security

```
# ✅ Version in path — clear, cache-friendly
GET /api/v1/users
GET /api/v2/users

# Deprecation headers on old versions
Deprecation: true
Sunset: Sat, 01 Jan 2026 00:00:00 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

**Checklist**:
- [ ] Old API versions still enforce current security controls (auth, rate limits)
- [ ] Deprecated endpoints have sunset date communicated
- [ ] Breaking changes never applied to live version without version bump

---

## 7. Webhook Security

Webhooks receive data from external services. They must verify the sender.

```js
// ❌ No signature verification — any request accepted
app.post('/webhooks/stripe', async (req, res) => {
  await processEvent(req.body);
});

// ✅ HMAC signature verification
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

app.post('/webhooks/stripe',
  express.raw({ type: 'application/json' }), // Raw body needed for sig verification
  async (req, res) => {
    const sig = req.headers['stripe-signature'];
    let event;
    try {
      event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET);
    } catch (err) {
      return res.status(400).json({ error: 'Invalid signature' });
    }
    await processEvent(event);
    res.json({ received: true });
  }
);
```

**Checklist**:
- [ ] All webhooks verify HMAC signature from provider
- [ ] Webhook secret stored in secrets manager (not hardcoded)
- [ ] Replay attack protection: `timestamp` in payload validated within tolerance window
- [ ] Idempotency: duplicate webhook delivery handled safely (check event ID)
- [ ] Webhook processing is async — return 200 immediately, process in background queue

---

## Output Format for Design Doc

```markdown
## API Security Controls

### Rate Limiting

| Endpoint | Limit | Window | Key |
|----------|-------|--------|-----|
| POST /auth/login | 5 | 15 min | IP |
| POST /api/orders | 10 | 1 min | user ID |
| GET /api/products | 200 | 1 min | IP |

### Validation

| Endpoint | Schema | Max Body Size |
|----------|--------|---------------|
| POST /api/orders | `CreateOrderSchema` (Zod) | 10KB |
| POST /api/upload | File type + size | 5MB |

### Response Controls
- Error messages: generic to client, detailed in logs
- `X-Powered-By` removed
- DTO pattern: explicit field allowlist on all responses
```
