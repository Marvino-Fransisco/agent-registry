# Browser Security Headers & CSP

## The Full Security Headers Stack

Every HTTP response from a web application should include these headers. Each prevents a distinct class of attack.

---

## 1. Content-Security-Policy (CSP)

CSP tells the browser which sources are trusted for each type of resource. It is the **primary defense** against XSS.

### CSP Directives Reference

| Directive | Controls | Example |
|-----------|---------|---------|
| `default-src` | Fallback for all resource types | `'self'` |
| `script-src` | JavaScript sources | `'self' 'nonce-{random}'` |
| `style-src` | CSS sources | `'self' 'nonce-{random}'` |
| `img-src` | Image sources | `'self' data: https://cdn.example.com` |
| `font-src` | Font sources | `'self' https://fonts.gstatic.com` |
| `connect-src` | XHR/Fetch/WebSocket targets | `'self' https://api.example.com` |
| `frame-src` | Allowed iframe sources | `'none'` |
| `object-src` | Flash/plugin sources | `'none'` |
| `base-uri` | Restricts `<base>` tag | `'self'` |
| `form-action` | Where forms can submit | `'self'` |
| `upgrade-insecure-requests` | Auto-upgrade HTTP→HTTPS | (no value) |

### Recommended Starting Policy

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{RANDOM_PER_REQUEST}';
  style-src 'self' 'nonce-{RANDOM_PER_REQUEST}';
  img-src 'self' data: https://cdn.yourdomain.com;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.yourdomain.com;
  frame-src 'none';
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
  report-uri /csp-violations;
```

### Nonce-Based CSP (Correct Approach)

```js
// Server generates a unique nonce per response
import crypto from 'crypto';

app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  res.setHeader('Content-Security-Policy',
    `script-src 'self' 'nonce-${res.locals.cspNonce}'; object-src 'none';`
  );
  next();
});
```

```html
<!-- Template uses the nonce -->
<script nonce="<%= cspNonce %>">
  // This script is allowed
</script>
```

### What to NEVER Use

```
# ❌ These defeat CSP entirely
script-src 'unsafe-inline'     # Allows all inline scripts = XSS protection gone
script-src 'unsafe-eval'       # Allows eval() = code injection risk
script-src *                   # Allows any domain
```

### CSP Rollout Strategy

Don't deploy a strict CSP cold — it will break things.

1. **Report-Only mode first**: `Content-Security-Policy-Report-Only`
   - Browser reports violations but doesn't block
   - Monitor `/csp-violations` for 1–2 weeks
2. **Fix violations** — inline scripts → nonce, inline styles → external CSS, etc.
3. **Switch to enforcement**: `Content-Security-Policy`
4. **Keep `report-uri`** — ongoing visibility into violations and attacks

---

## 2. Strict-Transport-Security (HSTS)

Forces HTTPS. Once sent, the browser never makes HTTP requests to this domain.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

| Parameter | Meaning |
|-----------|---------|
| `max-age=31536000` | 1 year — browser caches this |
| `includeSubDomains` | Applies to all subdomains |
| `preload` | Submit to browser preload list (hardcoded HTTPS) |

**Checklist**:
- [ ] HSTS set on all HTTPS responses
- [ ] `max-age` ≥ 31536000 (1 year) for production
- [ ] `includeSubDomains` included (prevents cookie theft via subdomain)
- [ ] Never set HSTS on non-HTTPS response (browser ignores it, but it signals confusion)

---

## 3. X-Frame-Options

Prevents clickjacking — your page being embedded in an attacker's iframe.

```
X-Frame-Options: DENY
# or
X-Frame-Options: SAMEORIGIN
```

**Modern equivalent via CSP** (more flexible):
```
Content-Security-Policy: frame-ancestors 'none';
# or
Content-Security-Policy: frame-ancestors 'self';
```

Use `frame-ancestors` in CSP if you already have CSP — it supersedes `X-Frame-Options` in modern browsers. Keep both for compatibility.

---

## 4. X-Content-Type-Options

Prevents MIME-sniffing — browser treating a `.jpg` response as JavaScript.

```
X-Content-Type-Options: nosniff
```

Always set this. No exceptions.

---

## 5. Referrer-Policy

Controls what URL is sent in the `Referer` header when navigating away from your site.

```
Referrer-Policy: strict-origin-when-cross-origin
```

| Policy | What is sent cross-origin |
|--------|--------------------------|
| `no-referrer` | Nothing |
| `strict-origin` | Origin only (https://example.com) |
| `strict-origin-when-cross-origin` | Full URL same-origin, origin only cross-origin ✅ recommended |
| `unsafe-url` | Full URL always ❌ leaks sensitive paths |

---

## 6. Permissions-Policy

Controls which browser APIs the page can use. Replaces Feature-Policy.

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(self)
```

Disable everything you don't use. This limits attack surface from XSS (attacker can't turn on camera).

---

## 7. CORS (Cross-Origin Resource Sharing)

CORS controls which origins can make credentialed requests to your API.

```js
// ❌ Never for credentialed APIs
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
// The spec actually forbids this combination, but many frameworks allow it incorrectly

// ✅ Explicit allowlist
const ALLOWED_ORIGINS = ['https://app.example.com', 'https://staging.example.com'];

app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (ALLOWED_ORIGINS.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
  }
  next();
});
```

**Checklist**:
- [ ] `Access-Control-Allow-Origin` uses explicit allowlist, never `*` for credentialed endpoints
- [ ] Preflight (`OPTIONS`) responses include correct `Access-Control-Allow-Methods`
- [ ] `Access-Control-Allow-Headers` only lists headers you actually use
- [ ] CORS allowlist validated from config — not derived from request `Origin` header without validation

---

## Full Header Audit Checklist

```bash
# Test your headers quickly
curl -I https://yourdomain.com | grep -E "strict-transport|content-security|x-frame|x-content-type|referrer|permissions"

# Or use: https://securityheaders.com
```

| Header | Required | Recommended Value |
|--------|----------|------------------|
| `Strict-Transport-Security` | ✅ Yes | `max-age=31536000; includeSubDomains` |
| `Content-Security-Policy` | ✅ Yes | See policy above |
| `X-Frame-Options` | ✅ Yes | `DENY` or `SAMEORIGIN` |
| `X-Content-Type-Options` | ✅ Yes | `nosniff` |
| `Referrer-Policy` | ✅ Yes | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | ✅ Yes | Deny unused APIs |
| `Cache-Control` (for sensitive pages) | ✅ Yes | `no-store, max-age=0` |

---

## Implementation by Framework

### Express.js — use `helmet`
```js
import helmet from 'helmet';
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    }
  }
}));
```

### Next.js — `next.config.js`
```js
const securityHeaders = [
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
];
module.exports = { async headers() { return [{ source: '/(.*)', headers: securityHeaders }] } };
```

### Nginx
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'; object-src 'none';" always;
```

---

## Output Format for Design Doc

```markdown
## Security Headers Configuration

**CSP Mode**: Enforcement (was Report-Only for 2 weeks prior)  
**CSP Nonce**: Injected per-request server-side

| Header | Value | Status |
|--------|-------|--------|
| Content-Security-Policy | `default-src 'self'; script-src 'self' 'nonce-...'; object-src 'none'` | ✅ |
| HSTS | `max-age=31536000; includeSubDomains` | ✅ |
| X-Frame-Options | `DENY` | ✅ |
| X-Content-Type-Options | `nosniff` | ✅ |
| Referrer-Policy | `strict-origin-when-cross-origin` | ✅ |

**CSP Violations Endpoint**: POST /csp-violations → logged to [service]
```
