# Research: API Exploration

## Purpose
Map a third-party API's capabilities, constraints, and integration cost before the team builds against it. Prevent discovering rate limits, missing endpoints, or auth complexity after integration has started.

## Exploration checklist

### 1. Authentication & authorization
- [ ] Auth method: API key / OAuth 2.0 / JWT / mTLS / HMAC signature?
- [ ] Token lifetime and refresh mechanism
- [ ] Scopes required for the needed operations
- [ ] IP allowlisting required?
- [ ] Sandbox / test environment available?

```bash
# Inspect auth flow from docs or OpenAPI spec
curl -s "https://api.example.com/openapi.json" | jq '.components.securitySchemes'
```

### 2. Rate limits
- [ ] Requests per second / minute / hour / day
- [ ] Per-endpoint vs global limits
- [ ] Rate limit headers in response (`X-RateLimit-Remaining`, `Retry-After`)
- [ ] Burst allowance?
- [ ] Cost at expected volume (if paid API)

```bash
# Check rate limit headers from a live call
curl -I -H "Authorization: Bearer <token>" "https://api.example.com/v1/resource" \
  | grep -i "rate\|limit\|retry\|quota"
```

### 3. Endpoint coverage
Map what the project needs vs what the API provides:

| Feature needed | API endpoint | Available? | Notes |
|----------------|-------------|-----------|-------|
| Create resource | POST /resources | ✅ | Requires `write` scope |
| Bulk create | POST /resources/batch | ⚠️ | Max 100 per batch |
| Real-time updates | WebSocket / webhooks | ✅ | Webhooks only, polling fallback |
| Delete | DELETE /resources/:id | ❌ | Soft delete only via PATCH |

### 4. Response structure
```bash
# Fetch a real response and inspect shape
curl -s -H "Authorization: Bearer <token>" "https://api.example.com/v1/resource/1" | jq '.'

# Check pagination model
curl -s "https://api.example.com/v1/resources?page=1&limit=20" | jq '{
  pagination: {has_next: .meta.has_next, total: .meta.total, cursor: .meta.next_cursor}
}'
```

Document: cursor-based vs offset pagination, max page size, response envelope structure.

### 5. Error handling
```bash
# Trigger a known error to inspect error envelope
curl -s -w "\n%{http_code}" "https://api.example.com/v1/resource/does-not-exist" | tail -2
```

Document:
- Error response format (`{"error": {...}}` vs `{"message": "..."}` vs RFC 7807 Problem Details)
- Which errors are retryable (5xx, 429) vs permanent (4xx)
- Whether error codes are documented or opaque strings

### 6. Idempotency
- Does the API support idempotency keys for write operations?
- What happens on duplicate requests (duplicate charge? duplicate record? 409 conflict?)
- Is there a request deduplication window?

### 7. Webhooks (if applicable)
- [ ] Payload format and schema
- [ ] Signature verification method (HMAC-SHA256 `X-Signature` header?)
- [ ] Retry policy (how many retries, backoff interval)
- [ ] Event types available
- [ ] Delivery order guarantee?

### 8. SDK & official client libraries
```bash
# Check if an official SDK exists and its maintenance status
npm view @company/sdk version time.modified
pip show company-sdk
```

Evaluate: official vs community, type safety, abstraction quality.

## Integration complexity scoring

| Factor | Low (1) | Medium (2) | High (3) |
|--------|---------|-----------|---------|
| Auth complexity | API key | OAuth 2.0 | mTLS / signed requests |
| Rate limit headroom | 10x our volume | 2–3x | < 2x |
| Pagination | Simple offset | Cursor | No pagination (full dump) |
| Webhook reliability | Retry + signature | Retry only | No webhook |
| SDK quality | Official, typed | Community | None / unmaintained |

Score 5–8 = Low integration risk | 9–12 = Medium | 13–15 = High

## Output format
Produce an API profile doc with:
- **API overview** (what it does, who operates it, SLA if published)
- **Auth summary** with example implementation snippet
- **Rate limit table**
- **Endpoint coverage matrix**
- **Error handling notes**
- **Integration complexity score**
- **Open questions** (things not findable in docs that require a sandbox test or vendor contact)

## Constraints
- Never assume rate limits from docs are accurate without testing in sandbox — they are often stale
- Always test error responses, not just happy paths
- Flag if the API has no versioning strategy — breaking changes with no notice are a high risk
- Note if the API uses REST, GraphQL, gRPC — each has different client complexity implications
