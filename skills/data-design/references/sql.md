# Query

## Query Standards

- Optimize when there is a **measurable performance issue**, not preemptively.
- Prefer the approach that is **most clear** (use `JOIN` or subqueries appropriately)
- Always explicitly select columns (**no** `SELECT *`)
- Always qualify columns when using multiple tables (use table aliases)
- Capital letter for syntaxes (`SELECT`, `FORM`, `WHERE`, etc.)
- Use consistent and meaningful table aliases (avoid random abbreviations)
- Never run `UPDATE` or `DELETE` without a `WHERE` clause
- Always validate write queries using a `SELECT` first
- Prioritize consistency and predictability over clever or complex solutions
- Follow the standard query structure below
  
  ```sql
  SELECT
    u.id,
    u.name,
    os.total_orders,
    os.total_spent,
    os.avg_order_value
  FROM
    users AS u
  JOIN (
    SELECT
      o.user_id,
      COUNT(o.id) AS total_orders,
      SUM(o.total_amount) AS total_spent,
      AVG(o.total_amount) AS avg_order_value
    FROM
      orders AS o
    WHERE
      o.status = 'completed'
      AND o.created_at >= (
        SELECT
          MIN(created_at)
        FROM
          orders
        WHERE
          status = 'completed'
      )
    GROUP BY
      o.user_id
  ) AS os ON os.user_id = u.id
  WHERE
    u.status = 'active'
  ORDER BY
    os.total_spent DESC;
  ```

---

## Instructions

### Step 1 - Identification

> Understand what the user needs before writing anything

- Identify the query intent:
  - (R) **Read** — SELECT, reporting, filtering, aggregation
  - (W) **Write** — INSERT, UPDATE, DELETE
- Identify whether a schema or existing query is provided
- If schema is provided, understand the relationships before writing
- If no schema is provided, infer reasonable table/column names and state assumptions clearly

---

### Step 2 - Validate Write Intent (W only)

> Never write blind

- Before producing any UPDATE or DELETE, produce a SELECT version first
- Clearly show the SELECT, then show the write query below it
- State explicitly what rows will be affected

---

### Step 3 - Write the Query

> Apply standards, write clearly

- Follow the standard query structure from Query Standards
- Always use explicit column names, never SELECT *
- Always qualify columns with table aliases when multiple tables are involved
- Use JOIN or subquery based on what is clearest, not what is cleverest
- Only optimize if the user has stated a performance concern

---

### Step 4 - Output

> Response and explanation

- Provide the final query
- Briefly explain the approach taken (why JOIN vs subquery, why this filter, etc.)
- If assumptions were made about schema, state them
- If it's a write query, show the SELECT validation first

---

## Edge Cases

### 1. N+1 Query Pattern

> Fetching related data in a loop instead of a single query

**Example:**

- Fetching all orders, then querying each order's items one by one in application code

**Practical Trade-off:**

- Always prefer a single JOIN or subquery over multiple round trips
- Use aggregation (STRING_AGG, JSON_AGG) to collapse related rows when needed

---

### 2. Filtering on Nullable Columns

> Using = or != on NULL columns produces unexpected results

**Example:**

- `WHERE deleted_at = NULL` returns nothing
- Correct: `WHERE deleted_at IS NULL`

**Practical Trade-off:**

- Always use IS NULL / IS NOT NULL for nullable columns
- Be explicit about soft delete filtering — always include `WHERE deleted_at IS NULL` unless historical data is intentionally needed

---

### 3. LIKE with Leading Wildcard

> A leading wildcard disables index usage

**Example:**

- `WHERE name LIKE '%john'` — full table scan
- `WHERE name LIKE 'john%'` — index can be used

**Practical Trade-off:**

- Avoid leading wildcards on large tables
- Use full-text search if prefix matching is insufficient

---

### 4. Aggregation Without Considering NULLs

> Aggregate functions silently ignore NULLs

**Example:**

- `AVG(rating)` skips NULL rows — result may be misleading
- `COUNT(column)` vs `COUNT(*)` behave differently on NULLs

**Practical Trade-off:**

- Use COALESCE when NULLs should be treated as a value (e.g., 0)
- Always be explicit about which COUNT behavior you need

---

### 5. UPDATE or DELETE Without WHERE

> Affects every row in the table

**Practical Trade-off:**

- Always validate with a SELECT first
- Never skip the WHERE clause, even in development

---

### 6. Implicit vs Explicit JOIN

> Old-style comma joins are harder to read and error-prone

**Example:**

- Implicit: `FROM orders o, users u WHERE o.user_id = u.user_id`
- Explicit: `FROM orders AS o JOIN users AS u ON o.user_id = u.user_id`

**Practical Trade-off:**

- Always use explicit JOIN syntax
- Easier to read, easier to extend, less risk of accidental cartesian product

---

### 7. Pagination on Large Tables

> OFFSET becomes slow on large datasets

**Example:**

- `LIMIT 20 OFFSET 10000` — database still scans 10,020 rows

**Practical Trade-off:**

- Use keyset pagination (cursor-based) for large tables
- `WHERE created_at < :last_cursor ORDER BY created_at DESC LIMIT 20`

---

## Bad Example

### 1. SELECT *

```sql
SELECT * FROM orders;
```

### 2. Unqualified Columns on Multiple Tables

```sql
SELECT id, name, created_at
FROM orders
JOIN users ON user_id = user_id;
```

### 3. Filtering NULL with =

```sql
SELECT * FROM users WHERE deleted_at = NULL;
```

### 4. UPDATE Without WHERE

```sql
UPDATE users SET is_active = false;
```

### 5. Leading Wildcard on Large Table

```sql
SELECT * FROM products WHERE name LIKE '%phone';
```

### 6. Implicit JOIN

```sql
SELECT o.order_id, u.full_name
FROM orders o, users u
WHERE o.user_id = u.user_id;
```

### 7. No SELECT Validation Before DELETE

```sql
DELETE FROM orders WHERE created_at < '2023-01-01';
```

---

## Good Example

### 1. Explicit Columns with Aliases

```sql
SELECT
  o.order_id,
  u.full_name,
  o.total_price,
  o.created_at
FROM
  orders AS o
JOIN
  users AS u ON o.user_id = u.user_id
WHERE
  o.deleted_at IS NULL
ORDER BY
  o.created_at DESC;
```

### 2. Filtering Nullable Columns Correctly

```sql
SELECT
  u.user_id,
  u.full_name
FROM
  users AS u
WHERE
  u.deleted_at IS NULL;
```

### 3. Validate Before DELETE

```sql
-- Step 1: Validate affected rows
SELECT
  o.order_id,
  o.created_at
FROM
  orders AS o
WHERE
  o.created_at < '2023-01-01'
  AND o.deleted_at IS NULL;

-- Step 2: Delete after validation
DELETE FROM orders
WHERE
  created_at < '2023-01-01'
  AND deleted_at IS NULL;
```

### 4. Aggregation with NULL Awareness

```sql
SELECT
  p.product_id,
  p.name,
  COALESCE(AVG(r.rating), 0) AS avg_rating,
  COUNT(r.review_id) AS total_reviews
FROM
  products AS p
LEFT JOIN
  reviews AS r ON p.product_id = r.product_id
WHERE
  p.deleted_at IS NULL
GROUP BY
  p.product_id,
  p.name
ORDER BY
  avg_rating DESC;
```

### 5. Cursor-Based Pagination

```sql
SELECT
  o.order_id,
  o.total_price,
  o.created_at
FROM
  orders AS o
WHERE
  o.deleted_at IS NULL
  AND o.created_at < :last_cursor
ORDER BY
  o.created_at DESC
LIMIT 20;
```

### 6. Full Realistic Example

```sql
-- Validate first
SELECT
  o.order_id,
  u.full_name,
  u.email,
  SUM(oi.total_price) AS order_total,
  COUNT(oi.order_item_id) AS total_items
FROM
  orders AS o
JOIN
  users AS u ON o.user_id = u.user_id
JOIN
  order_items AS oi ON o.order_id = oi.order_id
WHERE
  o.deleted_at IS NULL
  AND u.deleted_at IS NULL
  AND o.created_at >= '2024-01-01'
GROUP BY
  o.order_id,
  u.full_name,
  u.email
ORDER BY
  o.created_at DESC;
```
