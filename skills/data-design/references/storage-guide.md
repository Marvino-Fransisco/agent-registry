# Storage Guide

## Storage Guide Standards

- Store data where it is **accessed best**, not where it is easiest to put
- Prefer structured columns when data is frequently **queried or filtered**
- Use `JSONB` when schema flexibility is needed and data is read **as a whole**
- Store files and blobs in **object storage**, reference them by URL in the database
- Choose the smallest data type that safely fits the range of values

---

## Instructions

### Step 1 - Identification

> Understand what kind of data is being stored

- Identify the data category:
  - (S) **Structured** — fixed schema, typed columns, frequently queried (e.g. users, orders)
  - (Semi) **Semi-structured** — flexible schema, read as a whole, occasionally filtered (e.g. metadata, settings, shipping address)
  - (U) **Unstructured / Binary** — files, images, documents, media
- Identify access pattern:
  - Read-heavy vs write-heavy
  - Filtered by specific columns vs retrieved as a whole
  - Single row lookup vs bulk scan

---

### Step 2 - Choose Storage Mechanism

> Map each data category to the right storage approach

#### (S) Structured

- Use typed columns (`VARCHAR`, `INT`, `DECIMAL`, `TIMESTAMP`, `BOOLEAN`, `UUID`)
- Normalize into separate tables when entities have their own lifecycle
- Use `JSONB` only when the schema varies per row and is not a primary filter target

#### (Semi) Semi-structured

- Use `JSONB` columns when:
  - Schema varies across rows
  - Data is read and written as a whole unit
  - Individual keys are rarely used in WHERE or JOIN conditions
- Use structured columns when:
  - Specific keys are frequently filtered or indexed
  - Data integrity constraints are needed

#### (U) Unstructured / Binary

- Store files in object storage (e.g. S3, GCS, Azure Blob)
- Store only the **URL or key** in the database
- Store file metadata (name, size, mime type) alongside the URL when needed

---

### Step 3 - Output

> Response and explanation about the storage design

- Clearly state which category each piece of data falls into (S), (Semi), or (U)
- Provide the resulting storage design in:
  - **SQL schema** with column types and storage choices explained
  - **Diagram or table** showing which data goes where
- A summary about the storage decisions based on access patterns

---

## Edge Cases

> Edge cases that involve choosing between storage approaches

### 1. JSONB vs Normalized Table

> Semi-structured data that could go either way

**Example:**

- User preferences or settings that vary per user
- Product attributes that differ across categories

**Rule of Thumb:**

- Use `JSONB` when schema varies and data is read as a whole
- Use normalized table when individual fields are queried, filtered, or joined

**Practical Trade-off:**

- `JSONB` is simpler to evolve but harder to enforce constraints on
- Normalized table is stricter but enables better indexing and integrity

---

### 2. File and Binary Storage

> Storing user uploads, documents, or media

**Rule of Thumb:**

- Never store binary data directly in the database
- Store in object storage and keep a reference URL in the database

**Practical Trade-off:**

- Database bloat, backup cost, and query performance degrade with binary data
- Object storage is cheaper, faster for large files, and serves via CDN

---

### 3. Large Text and Documents

> Storing long-form content like articles, descriptions, or logs

**Rule of Thumb:**

- Use `TEXT` for unbounded content
- Use `VARCHAR(n)` only when there is a real business max length
- Store very large documents (megabytes) in object storage with a DB reference

**Practical Trade-off:**

- `TEXT` and `VARCHAR` have the same performance in PostgreSQL
- Very large values in DB rows increase table size and slow sequential scans

---

### 4. Enum vs Lookup Table vs VARCHAR

> Storing a fixed set of status values or categories

**Rule of Thumb:**

- Use `VARCHAR` with a `CHECK` constraint when the list is small and rarely changes
- Use a lookup table when the list is managed by the business and may grow
- Avoid native `ENUM` types — they are hard to alter and migrate

**Practical Trade-off:**

- `VARCHAR` + `CHECK` is simple but requires a migration to add values
- Lookup table is flexible but adds a `JOIN` to every query
- Native `ENUM` is fast but altering values requires `ALTER TYPE` which can lock the table

---

### 5. Caching Hot Data

> Frequently accessed data that is expensive to compute or query

**Example:**

- A product catalog summary shown on every page load
- A user's notification count

**Rule of Thumb:**

- Cache at the application layer (Redis, Memcached) when read frequency is high and data changes infrequently
- Denormalized summary tables can serve as a DB-level cache

**Practical Trade-off:**

- Cache introduces consistency lag — stale data is a real risk
- Only cache when there is a measurable read bottleneck
- Denormalized tables are easier to keep consistent but require write-time maintenance

---

## Bad Example

### 1. Storing Files as Base64 in Database

```sql
CREATE TABLE user_uploads (
  upload_id UUID PRIMARY KEY,
  file_base64 TEXT NOT NULL,
  uploaded_at TIMESTAMP DEFAULT now()
);
```

### 2. Overusing JSONB for Queryable Data

```sql
CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  data JSONB NOT NULL
);

-- data contains: {"name": "...", "price": 100, "category": "...", "stock": 5}
-- But every query filters on price and category
```

### 3. Wrong Data Type Choice

```sql
CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  price VARCHAR NOT NULL,
  stock_count VARCHAR NOT NULL
);
```

### 4. Native ENUM for Business-Managed Values

```sql
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered');

CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  status order_status NOT NULL
);

-- Adding 'cancelled' requires ALTER TYPE ... ADD VALUE
-- Cannot be done inside a transaction in some databases
```

---

## Good Example

### 1. File URL Reference Pattern

```sql
CREATE TABLE user_uploads (
  upload_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  file_url VARCHAR NOT NULL,
  file_name VARCHAR NOT NULL,
  file_size BIGINT,
  mime_type VARCHAR,
  uploaded_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

### 2. JSONB for Flexible Metadata (Read as a Whole)

```sql
CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  shipping_address JSONB NOT NULL,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- shipping_address: {"street": "...", "city": "...", "zip": "...", "country": "..."}
-- metadata: {"gift_wrap": true, "notes": "Leave at door"}
-- Read as a whole, not filtered by individual keys
```

### 3. Correct Data Types

```sql
CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  name VARCHAR NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock_count INT NOT NULL DEFAULT 0,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

### 4. VARCHAR with CHECK for Fixed Status Values

```sql
CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  status VARCHAR(20) NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

### 5. Lookup Table for Business-Managed Categories

```sql
CREATE TABLE categories (
  category_id UUID PRIMARY KEY,
  slug VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  category_id UUID REFERENCES categories(category_id),
  name VARCHAR NOT NULL,
  price DECIMAL NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);
```

### 6. Full Realistic Example

```sql
-- Users
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  full_name VARCHAR NOT NULL,
  preferences JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Categories (lookup table, business-managed)
CREATE TABLE categories (
  category_id UUID PRIMARY KEY,
  slug VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Products (structured columns for queryable fields)
CREATE TABLE products (
  product_id UUID PRIMARY KEY,
  category_id UUID REFERENCES categories(category_id),
  slug VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock_count INT NOT NULL DEFAULT 0,
  attributes JSONB,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Product Images (file references, not binary data)
CREATE TABLE product_images (
  image_id UUID PRIMARY KEY,
  product_id UUID NOT NULL REFERENCES products(product_id),
  image_url VARCHAR NOT NULL,
  alt_text VARCHAR,
  sort_order INT NOT NULL DEFAULT 0,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Orders (with snapshotting and CHECK constraint for status)
CREATE TABLE orders (
  order_id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(user_id),
  status VARCHAR(20) NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
  shipping_address JSONB NOT NULL,
  total_price DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMP DEFAULT now(),
  deleted_at TIMESTAMP
);

-- Order Items (with derived fields for audit)
CREATE TABLE order_items (
  order_item_id UUID PRIMARY KEY,
  order_id UUID NOT NULL REFERENCES orders(order_id),
  product_id UUID NOT NULL REFERENCES products(product_id),
  product_name VARCHAR NOT NULL,
  quantity INT NOT NULL,
  unit_price DECIMAL(10, 2) NOT NULL,
  total_price DECIMAL(10, 2) NOT NULL
);
```
