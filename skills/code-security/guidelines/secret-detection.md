# Secret Detection & Prevention

## What Counts as a Secret

Any value that grants access, proves identity, or unlocks a capability:

- API keys (Stripe, AWS, Twilio, SendGrid, etc.)
- Database connection strings with credentials
- JWT signing secrets
- OAuth client secrets
- Private keys (RSA, EC, SSH)
- Encryption keys / symmetric secrets
- Service account credentials (JSON key files)
- Webhook signing secrets
- Internal tokens between services
- SMTP credentials

**Rule**: If rotating this value would break something, it's a secret.

---

## Detection — Scanning the Codebase

### `gitleaks` (recommended — scans git history)

```bash
# Install
brew install gitleaks
# or
go install github.com/gitleaks/gitleaks/v8@latest

# Scan entire git history
gitleaks detect --source . --verbose

# Scan staged changes only (for pre-commit hook)
gitleaks protect --staged --verbose

# Generate report
gitleaks detect --source . --report-format json --report-path leaks.json
```

### `truffleHog` (deep entropy + regex scanning)

```bash
pip install trufflehog
trufflehog git file://. --only-verified
```

### `detect-secrets` (Yelp — good for CI)

```bash
pip install detect-secrets
detect-secrets scan > .secrets.baseline
detect-secrets audit .secrets.baseline
```

---

## Common Patterns to Flag

When manually reviewing code, flag any line matching:

| Pattern | Example |
|---------|---------|
| String assignment with key-like name | `const apiKey = "sk_live_..."` |
| AWS access key format | `AKIA[0-9A-Z]{16}` |
| Base64-encoded credential in config | `password: "dmFsdWU="` |
| `Authorization` header hardcoded | `headers: { Authorization: "Bearer eyJ..." }` |
| Connection string with credentials | `postgresql://user:pass@host/db` |
| `.pem` or private key content | `-----BEGIN RSA PRIVATE KEY-----` |
| `secret`, `password`, `token` assigned a non-env value | `SECRET_KEY = "my_hard_coded_secret"` |

---

## Remediation — Secret Found in Code

### Step 1: Rotate First, Remove Second

**Rotating before removing is critical.** If a secret has been committed, it must be treated as compromised — even if the commit was immediately reverted.

1. **Rotate/revoke** the exposed secret in the provider (AWS IAM, Stripe dashboard, etc.)
2. Generate a new secret
3. Update the secret in your secrets manager
4. Then clean the git history

### Step 2: Remove from Git History

```bash
# Using git-filter-repo (recommended over BFG for newer repos)
pip install git-filter-repo

# Remove file containing secret
git filter-repo --path path/to/secret-file.env --invert-paths

# Replace specific string
git filter-repo --replace-text <(echo "sk_live_abc123==>REDACTED")

# Force push all branches (coordinate with team)
git push origin --force --all
git push origin --force --tags
```

**Important**: After a force push, all collaborators must re-clone. Forks retain the history — if the repo is public, the secret must be treated as permanently compromised regardless of history rewrite.

### Step 3: Notify

- If the secret was in a public repository at any point, notify the affected service and rotate
- For AWS keys: check CloudTrail for unauthorized usage during the exposure window
- For Stripe/payment keys: check for unauthorized transactions

---

## Prevention — Never Committed in the First Place

### Environment Variables

```bash
# .env (NEVER commit this)
DATABASE_URL=postgresql://user:pass@localhost/mydb
STRIPE_SECRET_KEY=sk_live_...
JWT_SECRET=super-secret-value

# .env.example (COMMIT this — no real values)
DATABASE_URL=postgresql://user:pass@localhost/mydb
STRIPE_SECRET_KEY=sk_live_REPLACE_ME
JWT_SECRET=REPLACE_ME_WITH_256_BIT_RANDOM
```

```js
// Application code — always read from environment
const stripeKey = process.env.STRIPE_SECRET_KEY;
if (!stripeKey) throw new Error('STRIPE_SECRET_KEY not set');
```

**Checklist**:
- [ ] `.env` in `.gitignore`
- [ ] `.env.example` committed with placeholder values only
- [ ] No defaults fallback to real credentials: `process.env.DB_PASS || 'admin'`

---

### Secrets Managers (Production)

Prefer dedicated secrets managers over environment variables for production:

| Platform | Service |
|----------|---------|
| AWS | Secrets Manager, Parameter Store (SecureString) |
| GCP | Secret Manager |
| Azure | Key Vault |
| HashiCorp | Vault |
| Self-hosted | Infisical, Doppler |

```js
// AWS Secrets Manager example
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";

const client = new SecretsManagerClient({ region: "us-east-1" });
const { SecretString } = await client.send(
  new GetSecretValueCommand({ SecretId: "prod/myapp/stripe" })
);
const { stripeKey } = JSON.parse(SecretString);
```

**Checklist**:
- [ ] Production secrets in secrets manager — not `.env` files on servers
- [ ] IAM/RBAC on secrets manager: only the service that needs it can read it
- [ ] Secret access logged and alertable
- [ ] Rotation policy defined and automated where possible

---

### Pre-commit Hooks

```bash
# Install pre-commit
pip install pre-commit

# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.2
    hooks:
      - id: gitleaks
```

```bash
pre-commit install
```

This blocks commits that contain detected secrets before they reach the repo.

---

### CI/CD Secret Scanning

```yaml
# GitHub Actions
- name: Secret Scan
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

GitHub also has native secret scanning for public repos (and private with Advanced Security).

---

## Output Format for Design Doc

```markdown
## Secret Management Plan

**Scanning**: gitleaks in pre-commit + CI pipeline  
**Storage**: [env vars for local / AWS Secrets Manager for prod]  
**Rotation Policy**:

| Secret | Rotation Frequency | Owner |
|--------|-------------------|-------|
| Stripe API key | On compromise / quarterly | Platform team |
| JWT signing secret | Quarterly | Auth team |
| DB credentials | 90 days | Infra team |

**Incident Response**:
1. Rotate immediately
2. Check access logs for unauthorized use
3. Clean git history if in version control
4. Post-mortem within 48h
```
