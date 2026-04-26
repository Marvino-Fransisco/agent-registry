# Schema Design

## Schema Design Standards

- Performance first normalization second
- Performance > 3NF
- Normalization is a **tool** not a **goal**

---

## Instructions

### Step 1 - Identification

> Identify whether any data is provided

- Identify whether the user has provided:
  - (A) data or data structure (JSON, Type, Interface, Purchase Note, Unromalized Table, etc...)
  - (B) No data, and instead is requesting a schema design from scratch
- Identify and understand the current schema design

---

### Step 2 - Create Schema

> Schema creation process based on condition (A) or (B)

#### (A) Data is Provided

- Extract all relevant fields, entities, and relationships from user's input
- Infer missing structure where necessary
- Normalize and refine into a clean schema design

#### (B) No Data

- Identify the user's intent
- Create a clean schema design
- **OPTIONAL** - Ask about the business process around the data that might be needed by user but they are not realized

---

### Step 3 - Denormalization

> Based on the normalization schema and user's intent detect should denormalization table created

- Identify read heavy table that needs lot of join and sub queries based on schema design against user's intent
- If there are any, then create the denormalized table

---

### Step 4 - Output

> Your response and explanation about the schema design

- Clearly state which path was chosen (A) or (B)
- Provide the resulting schema design in:
  - **table**
  - **ERD** via `Mermaid` diagram
- A summary about the schema design based on user's intent

---

## Edge Cases

> Edge cases that could be use with the trade offs of performance

### 1. Multi-valued attributes

> Attributes that naturally contain multiple values for a single entity

**Example:**

- A user with multiple phone numbers

**Normalization Rule:**

- In strict First Normal Form (1NF), multi-valued attributes must be separated into a child table

**Practical Trade-off:**

- Can be stored as a JSON array when:
- Read performance is prioritized
- The data is not frequently queried individually
- Simplicity is preferred over strict normalization

---

### 2. Derived / Computer Fields

> Storing computed fields for performance, historical accuracy, and audit reasons

**Example:**

- `total_price = quantity × price`

**Normalization Rule:**

- Never store computed or calculation value

**Practical Trade-off:**

- Read performance is prioritized
- Historical and audit reasons

### 3. Time-dependent Data (Historical Truth vs Current Truth)

> Storing historical truth

**Example:**

- User changes address
- Order should still keep the old address

**Normalization Rule:**

- Store address in one place

**Practical Trade-off:**

- Duplicate data **on purpose** (Snapshotting)
- Historical data still hold the trugth

---

### 4. Many-to-many with extra meaning

> A many-to-many relationship that carries additional attributes, making it more than just a simple association
> This one is often under-engineer

**Example:**

- `User ↔ Project`
- But also includes:
  - role (e.g., admin, contributor)
  - joined_at
  - permissions

**Practical Trade-off:**

- Storing metadata to create context in a junction table

---

### 5. Natural Key vs Surrogate Key Conflicts

> Natural keys (like email) may change over time, which can break relationships and violate the assumption that primary keys are stable.
> Surrogate keys (like user_id) provide stability, but require additional constraints (e.g. UNIQUE) to enforce real-world rules.

**Example:**

- `email` as Primary Key -> User change `email` -> Mess and update everywhere
- `user_id` as Primary Key -> Stable but "hide" business meaning

**Practical Trade-off:**

- Use surrogate key as PK
- Use natural key as UNIQUE constraint

---

### 6. Performance-driven denormalization

> A perfectly normalized schema is often too slow for real systems

**Example:**

- A data that need to join 5 tables
- Heavy resource usage

**Practical Trade-off:**

- Break normalization intentionally:
  - Duplicate columns
  - Pre-join tables
  - Caching aggregates

---

### 7. Ambigous Entity Boundaries

> Is an attribute is an entity or just a field?
> This part is often over-normalized and under-engineered
> No strict rule - depends on the usage

**Example:**

- Is "address" an entity or just a field?
- Is "country" an entity or just a field?

---

## Bad Example

### 1. Exposing DB Column Names in API Response

```json
{
  "user_id": 1,
  "created_at": "2024-01-01",
  "is_deleted": false
}
```

### 2. Over-Normalized Many-to-Many (Under-Engineered Junction Table)

```sql
CREATE TABLE user_project (
  user_id UUID,
  project_id UUID
);
```

### 3. Natural Key as Primary Key

```sql
CREATE TABLE users (
  email VARCHAR PRIMARY KEY,
  full_name VARCHAR
);
```

### 4. Over-Normalized Ambiguous Entity

```sql
CREATE TABLE countries (id UUID PRIMARY KEY, name VARCHAR, code VARCHAR);
CREATE TABLE cities (id UUID PRIMARY KEY, country_id UUID, name VARCHAR);
CREATE TABLE addresses (id UUID PRIMARY KEY, city_id UUID, street VARCHAR);
CREATE TABLE users (id UUID PRIMARY KEY, address_id UUID);
```

### 5. Missing Derived Field on Historical Data

```sql
CREATE TABLE order_items (
  id UUID PRIMARY KEY,
  quantity INT,
  unit_price DECIMAL
);
```

---

## Good Example

### 1. Proper Casing with API Contract Separation

```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  full_name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

```json
{
  "userId": "abc-123",
  "fullName": "John Doe",
  "createdAt": "2024-01-01"
}
```

### 2. Many-to-Many with Context (Junction Table Done Right)

```sql
CREATE TABLE user_project (
  user_id UUID REFERENCES users(user_id),
  project_id UUID REFERENCES projects(project_id),
  role VARCHAR NOT NULL,
  permissions JSONB,
  joined_at TIMESTAMP DEFAULT now(),
  PRIMARY KEY (user_id, project_id)
);
```

### 3. Surrogate Key + Natural Key as Unique Constraint

```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  full_name VARCHAR NOT NULL
);
```

### 4. Snapshotting for Historical Truth

```sql
CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(user_id),
  shipping_address JSONB NOT NULL,
  unit_price DECIMAL NOT NULL,
  total_price DECIMAL NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

### 5. Denormalization for Read-Heavy Data

```sql
CREATE TABLE order_summary (
  order_id UUID PRIMARY KEY,
  user_id UUID,
  total_items INT,
  total_price DECIMAL,
  product_names TEXT[],
  updated_at TIMESTAMP DEFAULT now()
);
```

### 6. Full Realistic Example

```sql
-- Users
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  full_name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Products
CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  slug VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  price DECIMAL NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Orders (with snapshotting)
CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(user_id),
  shipping_address JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Order Items (with derived fields stored for audit)
CREATE TABLE order_items (
  order_item_id UUID PRIMARY KEY,
  order_id UUID REFERENCES orders(order_id),
  product_id UUID REFERENCES products(product_id),
  product_name VARCHAR NOT NULL,
  quantity INT NOT NULL,
  unit_price DECIMAL NOT NULL,
  total_price DECIMAL NOT NULL
);

-- Denormalized Read Table
CREATE TABLE order_summary (
  order_id UUID PRIMARY KEY,
  user_id UUID,
  full_name VARCHAR,
  total_items INT,
  total_price DECIMAL,
  product_names TEXT[],
  updated_at TIMESTAMP DEFAULT now()
);
```
