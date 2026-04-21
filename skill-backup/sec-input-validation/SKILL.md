---
name: sec-input-validation
description: Review and design input validation, sanitization, and encoding controls for frontend and backend code. Use when auditing forms, API request bodies, query params, file uploads, or any user-supplied data. Trigger for any mention of XSS, injection, sanitization, validation, unsafe input, or when reviewing code that processes external data. Covers SQL injection, XSS, command injection, path traversal, prototype pollution, and mass assignment.
---

# Input Validation & Sanitization

## Core Principle

**Never trust input.** Every byte that enters your system from an external source — user forms, API requests, query params, headers, file contents, environment variables, webhook payloads — is adversarial until proven safe.

Validation and sanitization are different things:
- **Validation** — reject or accept. Does this match the expected shape?
- **Sanitization** — transform. Make unsafe input safe for a specific output context.
- **Encoding** — context-aware escaping. Prevents injection at the output layer.

All three are needed. None alone is sufficient.

---

## Injection Vulnerabilities

### SQL Injection

**Never** build queries with string concatenation.

```js
// ❌ Vulnerable
const q = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Safe — parameterized
const q = `SELECT * FROM users WHERE email = $1`;
db.query(q, [email]);
```

**ORM pitfalls** — ORMs are not automatically safe:
```js
// ❌ Still vulnerable — raw interpolation inside ORM
User.findAll({ where: sequelize.literal(`name = '${name}'`) });

// ✅ Safe
User.findAll({ where: { name } });
```

**Checklist**:
- [ ] All SQL uses parameterized queries or prepared statements
- [ ] No `sequelize.literal()`, `knex.raw()`, `$queryRaw` with interpolated input
- [ ] ORM `where` clauses never receive raw user strings
- [ ] Stored procedures parameterized (not dynamic SQL inside proc)

---

### NoSQL Injection

MongoDB, Firestore, and similar databases have injection risks too.

```js
// ❌ Vulnerable — user can pass { $gt: "" } to bypass check
User.findOne({ password: req.body.password });

// ✅ Safe — validate type before query
if (typeof req.body.password !== 'string') throw new Error('Invalid');
User.findOne({ password: req.body.password });
```

**Checklist**:
- [ ] All query fields validated as expected primitive types before use
- [ ] `mongoose-sanitize` or equivalent strips `$` operators from user input
- [ ] Aggregation pipelines with user data use `$match` with validated fields only

---

### Command Injection

```js
// ❌ Vulnerable
exec(`convert ${filename} output.png`);

// ✅ Safe — use execFile with argument array (no shell interpretation)
execFile('convert', [filename, 'output.png']);
```

**Checklist**:
- [ ] No `exec()`, `system()`, `eval()`, `popen()` with user input
- [ ] File path inputs sanitized to prevent path traversal (`../../etc/passwd`)
- [ ] Shell metacharacters (`; | & $ ` ' "`) stripped or rejected if shell calls unavoidable

---

### Path Traversal

```js
// ❌ Vulnerable
const filePath = path.join('/uploads', req.params.filename);
// User can pass: ../../etc/passwd

// ✅ Safe
const resolved = path.resolve('/uploads', req.params.filename);
if (!resolved.startsWith('/uploads')) throw new Error('Invalid path');
```

**Checklist**:
- [ ] All file paths resolved with `path.resolve()` and prefix-checked
- [ ] Uploaded filenames sanitized — strip `..`, `/`, null bytes
- [ ] File extension whitelisted (not just checked for absence of `.exe`)

---

## XSS (Cross-Site Scripting)

XSS occurs when user data is rendered as HTML/JS without proper encoding.

### DOM-based (Frontend)

```js
// ❌ Vulnerable
element.innerHTML = userInput;
document.write(userInput);

// ✅ Safe
element.textContent = userInput;
// Or use a sanitization library if HTML is required
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

**React / Vue**:
```jsx
// ❌ Vulnerable
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ Safe (only when HTML is truly needed)
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

### Stored XSS (Backend)

If user content is stored and later rendered as HTML:
- Sanitize at **render time**, not just at storage time
- Use a server-side sanitizer (`sanitize-html`, `bleach` for Python, `HtmlSanitizer` for C#)
- Allowlist permitted tags — do not denylist dangerous ones

**Checklist**:
- [ ] No `innerHTML`, `outerHTML`, `document.write` with unsanitized input
- [ ] `dangerouslySetInnerHTML` / `v-html` only used with DOMPurify output
- [ ] Server-rendered templates use auto-escaping (Jinja2, Blade, Handlebars all do by default)
- [ ] Rich text editors output sanitized HTML before storage and before render
- [ ] `href` attributes validated — `javascript:` URIs are XSS vectors

---

## Mass Assignment

```js
// ❌ Vulnerable — user can pass { role: "admin", isVerified: true }
User.update(req.body, { where: { id: req.user.id } });

// ✅ Safe — explicit allowlist
const { name, email } = req.body;
User.update({ name, email }, { where: { id: req.user.id } });
```

**Checklist**:
- [ ] No `Model.create(req.body)` or `Model.update(req.body)` directly
- [ ] Input mapped to explicit DTO / allowed field list before persistence
- [ ] Sensitive fields (`role`, `isAdmin`, `balance`) excluded from user-controlled updates

---

## Prototype Pollution (JavaScript)

```js
// ❌ Vulnerable — deep merge from user input
_.merge({}, JSON.parse(userInput));

// User sends: { "__proto__": { "isAdmin": true } }
```

**Checklist**:
- [ ] `JSON.parse()` output never deep-merged with `_.merge`, `Object.assign` recursively
- [ ] Use `Object.create(null)` for key-value stores that accept arbitrary user keys
- [ ] Libraries: use `lodash ≥ 4.17.21`, `qs ≥ 6.7.3` (both patch prototype pollution)

---

## File Upload Validation

```js
// ❌ Extension check only — trivially bypassed
if (!file.name.endsWith('.jpg')) throw new Error();

// ✅ Magic bytes check
import { fileTypeFromBuffer } from 'file-type';
const type = await fileTypeFromBuffer(buffer);
if (!['image/jpeg', 'image/png'].includes(type?.mime)) throw new Error('Invalid file type');
```

**Checklist**:
- [ ] MIME type verified via magic bytes, not just extension or `Content-Type` header
- [ ] File size limit enforced at server level (not just client-side)
- [ ] Uploads stored outside webroot — not directly served
- [ ] Filename sanitized before storage: strip special chars, use UUID as stored name
- [ ] Virus scanning for arbitrary file uploads (ClamAV, cloud-based scan)
- [ ] SVG files treated as HTML — sanitize or refuse

---

## Validation Libraries (by ecosystem)

| Ecosystem | Recommended Library |
|-----------|-------------------|
| Node.js | `zod`, `joi`, `yup` |
| Python | `pydantic`, `marshmallow`, `cerberus` |
| Go | `validator` (go-playground) |
| Java | Bean Validation (`jakarta.validation`) |
| PHP | Laravel Validation, `respect/validation` |
| Ruby | `dry-validation`, ActiveModel validations |

**Validation rules to always enforce**:
- Type (string, number, boolean — not "anything")
- Length / size bounds
- Format (regex for email, UUID, phone)
- Enum membership for categorical fields
- Numeric range (min, max, no negative IDs)

---

## Output Format for Design Doc

```markdown
## Input Validation Controls

| Input | Location | Validation Rule | Sanitization | Encoding |
|-------|----------|----------------|-------------|---------|
| email | POST /auth/login body | RFC 5322 format, max 254 chars | Trim whitespace | N/A — not rendered |
| comment | POST /comments body | String, max 1000 chars | DOMPurify at render | HTML-escaped in template |
| file | POST /upload | image/jpeg or image/png magic bytes, max 5MB | UUID rename | N/A — not served inline |
```
