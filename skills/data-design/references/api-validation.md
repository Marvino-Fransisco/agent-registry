# API Validation

## API Validation Standards

- Separate **database schema** from **API contract** — they serve different purposes
- Validate all input **before processing** — fail fast, fail clear
- Return **consistent error format** across all endpoints
- Map fields explicitly between DB and API — never auto-map or pass-through
- Use explicit allow-lists for updatable fields — never accept arbitrary keys

---

## Instructions

### Step 1 - Identification

> Understand what the API endpoint does

- Identify the endpoint intent:
  - (R) **Read** — GET, list, detail, search
  - (W) **Write** — POST, PUT, PATCH, DELETE
- Identify whether a schema, existing API contract, or both are provided
- Identify the data flow:
  - What the client sends (request body, query params, path params)
  - What the server stores (DB columns)
  - What the client receives (response body)

---

### Step 2 - Define Contract

> Define request shape, response shape, and validation rules

#### Request Shape (W only)

- Define required vs optional fields
- Define field types and constraints (length, range, format)
- Define allow-list of fields that can be created or updated

#### Response Shape

- Define what fields are returned
- Omit internal fields (e.g. `deleted_at`, internal IDs that have no meaning to the client)
- Decide pagination format for list endpoints

#### Validation Rules

- Required fields must be present and non-null
- String fields must have max length
- Numeric fields must have range constraints
- Enum-like fields must be validated against allowed values
- Nested objects must be validated recursively

---

### Step 3 - Map Fields

> Explicit mapping between DB columns and API fields

- Create a clear mapping: `db_column_name` ↔ `apiFieldName`
- Apply transformations:
  - `snake_case` → `camelCase`
  - Internal fields omitted from response
  - Computed or derived fields added to response
- Never expose:
  - `deleted_at` (soft delete is internal)
  - Internal foreign key IDs unless meaningful to the client
  - Database constraint names or error details

---

### Step 4 - Output

> Response and explanation about the API contract

- Clearly state which intent (R) or (W) the endpoint serves
- Provide:
  - **Request body** example (for writes)
  - **Response body** example
  - **Field mapping table** (DB column ↔ API field)
  - **Validation rules** summary
  - **Error response** format
- A summary about the API design decisions

---

## Edge Cases

> Edge cases involving API contract design and validation

### 1. Exposing DB Column Names

> Leaking internal schema details to the client

**Example:**

- API returns `user_id`, `created_at`, `is_deleted` directly from DB

**Rule of Thumb:**

- Always transform DB names to `camelCase` in API responses
- Never return internal fields like `deleted_at`, `is_active` flags

**Practical Trade-off:**

- Direct pass-through is faster to build but couples the client to the DB schema
- Any DB rename becomes a breaking API change

---

### 2. Over-fetching and Under-fetching

> Returning too much or too little data

**Example:**

- Returning full user object with all fields when only name and avatar are needed
- Requiring 3 API calls to assemble a single view

**Rule of Thumb:**

- Return only what the client needs for the given view
- Allow field selection or provide tailored endpoints for different contexts

**Practical Trade-off:**

- Over-fetching wastes bandwidth and exposes unnecessary data
- Under-fetching increases round trips and client complexity
- Design around the view, not around the table

---

### 3. Nested Object vs Flat Response

> Choosing response structure for related data

**Example:**

- Should an order response include the user as a nested object or just a `userId`?

**Rule of Thumb:**

- Nest when the client always needs the related data together
- Return only the ID when the client can fetch separately or already has the data

**Practical Trade-off:**

- Nested objects reduce round trips but increase response size
- Flat IDs keep responses small but may require additional requests
- Follow the principle: include what the consumer of this endpoint needs

---

### 4. Null vs Omit Empty Fields

> How to handle fields with no value in API responses

**Example:**

- A user with no phone number: `{"phoneNumber": null}` vs omitting the field entirely

**Rule of Thumb:**

- Be consistent — pick one strategy and stick to it across all endpoints
- Prefer returning `null` for optional fields to keep the response shape predictable

**Practical Trade-off:**

- Returning `null` makes the schema explicit — clients know the field exists
- Omitting fields reduces response size but makes the shape unpredictable
- Consistency matters more than which option is chosen

---

### 5. Error Response Leaking Internals

> Returning database errors, stack traces, or constraint names to the client

**Example:**

- `{"error": "duplicate key value violates unique constraint \"users_email_key\""}`

**Rule of Thumb:**

- Return a generic, user-friendly message
- Map known DB errors to specific API error codes
- Log the full error server-side for debugging

**Practical Trade-off:**

- Detailed errors help debugging but expose attack surface
- Generic errors are safer but less helpful during development
- Use error codes that the client can map to UI messages

---

### 6. Pagination Contract

> Choosing pagination approach for list endpoints

**Example:**

- Offset-based: `?page=2&limit=20`
- Cursor-based: `?cursor=abc123&limit=20`

**Rule of Thumb:**

- Use offset-based for small, stable datasets
- Use cursor-based for large or frequently changing datasets

**Practical Trade-off:**

- Offset-based is simpler but slow on large tables and may skip/duplicate rows on changes
- Cursor-based is consistent and fast but harder to implement and prevents random page access

---

### 7. File Upload Validation

> Accepting and validating file uploads through the API

**Example:**

- User uploads a profile image

**Rule of Thumb:**

- Validate file type (MIME), file size, and dimensions server-side
- Never trust client-side validation alone
- Return a URL reference after upload, store the file in object storage

**Practical Trade-off:**

- Strict validation prevents abuse but may reject valid edge cases
- Always set a max file size to prevent denial of service
- Validate MIME type by reading file headers, not just the extension

---

## Bad Example

### 1. DB Column Names in API Response

```json
{
  "user_id": "abc-123",
  "full_name": "John Doe",
  "created_at": "2024-01-01T00:00:00Z",
  "deleted_at": null,
  "is_active": true
}
```

### 2. Leaking Internal Errors

```json
{
  "error": "column \"emial\" does not exist",
  "stack": "at Query.run (pg/client.js:142:11)..."
}
```

### 3. No Input Validation on Write

```json
// POST /api/users
// No validation — accepts anything
{
  "name": "",
  "email": "not-an-email",
  "age": -5,
  "role": "superadmin"
}
```

### 4. Over-fetching (Returning Raw DB Row)

```json
// GET /api/users/abc-123
{
  "user_id": "abc-123",
  "email": "john@example.com",
  "full_name": "John Doe",
  "password_hash": "$2b$12$...",
  "reset_token": "abc123",
  "last_login_ip": "192.168.1.1",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-15T00:00:00Z",
  "deleted_at": null
}
```

### 5. Inconsistent Error Format

```json
// Endpoint A
{ "message": "User not found" }

// Endpoint B
{ "error": { "details": "Invalid input" } }

// Endpoint C
{ "status": 400, "reason": "Bad data" }
```

### 6. Accepting Arbitrary Fields

```json
// PATCH /api/users/abc-123
{
  "fullName": "New Name",
  "role": "admin",
  "isAdmin": true,
  "internalNotes": "bypassed"
}
```

---

## Good Example

### 1. Proper API Contract with Field Mapping

| DB Column | API Field | Notes |
| --------- | --------- | ----- |
| `user_id` | `userId` | transformed to camelCase |
| `full_name` | `fullName` | transformed to camelCase |
| `email` | `email` | same in both |
| `created_at` | `createdAt` | transformed to camelCase |
| `deleted_at` | *(omitted)* | internal field, not exposed |

```json
{
  "userId": "abc-123",
  "fullName": "John Doe",
  "email": "john@example.com",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### 2. Consistent Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      },
      {
        "field": "age",
        "message": "Must be a positive integer"
      }
    ]
  }
}
```

### 3. Input Validation on Write

```json
// POST /api/users — Request
{
  "fullName": "John Doe",
  "email": "john@example.com",
  "age": 25
}
```

Validation rules:

| Field | Type | Required | Constraints |
| ----- | ---- | -------- | ----------- |
| `fullName` | string | yes | min 1, max 100 |
| `email` | string | yes | valid email format |
| `age` | integer | no | min 0, max 150 |

```json
// POST /api/users — Response (201)
{
  "userId": "abc-123",
  "fullName": "John Doe",
  "email": "john@example.com",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### 4. Tailored Response (Not Over-fetching)

```json
// GET /api/users/abc-123
{
  "userId": "abc-123",
  "fullName": "John Doe",
  "email": "john@example.com",
  "avatarUrl": "https://cdn.example.com/avatars/abc-123.jpg",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### 5. Explicit Allow-List for Updates

```json
// PATCH /api/users/abc-123
// Only these fields are accepted:
{
  "fullName": "New Name"
}

// "role", "isAdmin", "internalNotes" are ignored or rejected
```

### 6. Offset-Based Pagination Response

```json
{
  "data": [
    {
      "orderId": "ord-001",
      "totalPrice": 150.00,
      "status": "delivered",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 153,
    "totalPages": 8
  }
}
```

### 7. Cursor-Based Pagination Response

```json
{
  "data": [
    {
      "orderId": "ord-001",
      "totalPrice": 150.00,
      "status": "delivered",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "nextCursor": "eyJvcmRlcl9pZCI6Im9yZC0wMjAifQ==",
    "hasMore": true,
    "limit": 20
  }
}
```

### 8. Full Realistic Example

#### Create Order — POST /api/orders

**Request:**

```json
{
  "items": [
    {
      "productId": "prod-001",
      "quantity": 2
    },
    {
      "productId": "prod-002",
      "quantity": 1
    }
  ],
  "shippingAddress": {
    "street": "123 Main St",
    "city": "Springfield",
    "zip": "62701",
    "country": "US"
  }
}
```

**Validation Rules:**

| Field | Type | Required | Constraints |
| ----- | ---- | -------- | ----------- |
| `items` | array | yes | min 1 item |
| `items[].productId` | string (UUID) | yes | must exist in DB |
| `items[].quantity` | integer | yes | min 1, max 100 |
| `shippingAddress` | object | yes | all sub-fields required |
| `shippingAddress.street` | string | yes | max 200 |
| `shippingAddress.city` | string | yes | max 100 |
| `shippingAddress.zip` | string | yes | max 20 |
| `shippingAddress.country` | string | yes | 2-letter ISO code |

**Field Mapping:**

| DB Column | API Field | Transform |
| --------- | --------- | --------- |
| `order_id` | `orderId` | camelCase |
| `user_id` | *(omitted)* | derived from auth context |
| `status` | `status` | same |
| `shipping_address` | `shippingAddress` | JSONB → camelCase keys |
| `total_price` | `totalPrice` | camelCase |
| `created_at` | `createdAt` | camelCase |
| `deleted_at` | *(omitted)* | internal |

**Response (201):**

```json
{
  "orderId": "ord-100",
  "status": "pending",
  "items": [
    {
      "productId": "prod-001",
      "productName": "Wireless Mouse",
      "quantity": 2,
      "unitPrice": 25.00,
      "totalPrice": 50.00
    },
    {
      "productId": "prod-002",
      "productName": "Keyboard",
      "quantity": 1,
      "unitPrice": 75.00,
      "totalPrice": 75.00
    }
  ],
  "shippingAddress": {
    "street": "123 Main St",
    "city": "Springfield",
    "zip": "62701",
    "country": "US"
  },
  "totalPrice": 125.00,
  "createdAt": "2024-06-15T14:30:00Z"
}
```

**Error Response (400):**

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": [
      {
        "field": "items[1].productId",
        "message": "Product not found"
      },
      {
        "field": "items[0].quantity",
        "message": "Must be between 1 and 100"
      }
    ]
  }
}
```
