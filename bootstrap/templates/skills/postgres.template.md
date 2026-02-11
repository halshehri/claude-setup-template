---
name: postgres
description: Apply PostgreSQL best practices when writing queries, designing schemas, or optimizing database operations. Use when working with PostgreSQL databases.
---

# PostgreSQL Best Practices

Apply these standards when working with PostgreSQL.

## Schema Design

### Naming Conventions
- **Tables**: `snake_case`, plural (e.g., `users`, `order_items`)
- **Columns**: `snake_case` (e.g., `created_at`, `user_id`)
- **Primary keys**: `id` (prefer UUID or BIGSERIAL)
- **Foreign keys**: `{referenced_table_singular}_id` (e.g., `user_id`)
- **Indexes**: `idx_{table}_{columns}` (e.g., `idx_users_email`)
- **Constraints**: `{table}_{type}_{columns}` (e.g., `users_unique_email`)

### Table Structure

```sql
-- Standard table template
CREATE TABLE users (
    -- Primary key
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Business columns
    email VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'user',
    status VARCHAR(20) NOT NULL DEFAULT 'active',

    -- Metadata
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Constraints
    CONSTRAINT users_unique_email UNIQUE (email),
    CONSTRAINT users_check_role CHECK (role IN ('user', 'admin', 'moderator')),
    CONSTRAINT users_check_status CHECK (status IN ('active', 'inactive', 'suspended'))
);

-- Always add indexes for foreign keys and frequently queried columns
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_status ON users (status) WHERE status = 'active';

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();
```

### Foreign Key Relationships

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Always index foreign keys
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_status ON orders (status);

CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_order_items_order_id ON order_items (order_id);
CREATE INDEX idx_order_items_product_id ON order_items (product_id);
```

## Query Patterns

### Basic CRUD

```sql
-- Insert with returning
INSERT INTO users (email, name, role)
VALUES ('john@example.com', 'John Doe', 'user')
RETURNING *;

-- Insert multiple
INSERT INTO users (email, name)
VALUES
    ('user1@example.com', 'User 1'),
    ('user2@example.com', 'User 2')
RETURNING *;

-- Update with returning
UPDATE users
SET name = 'Jane Doe', updated_at = NOW()
WHERE id = '123'
RETURNING *;

-- Upsert (insert or update)
INSERT INTO users (email, name)
VALUES ('john@example.com', 'John Doe')
ON CONFLICT (email)
DO UPDATE SET
    name = EXCLUDED.name,
    updated_at = NOW()
RETURNING *;

-- Soft delete pattern
UPDATE users
SET status = 'deleted', updated_at = NOW()
WHERE id = '123';
```

### Filtering and Pagination

```sql
-- Pagination with cursor (preferred for large datasets)
SELECT *
FROM users
WHERE created_at < :cursor_timestamp
ORDER BY created_at DESC
LIMIT :page_size;

-- Pagination with offset (simpler but slower for deep pages)
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT :page_size
OFFSET :offset;

-- Count with pagination
SELECT
    COUNT(*) OVER() AS total_count,
    *
FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT :page_size
OFFSET :offset;

-- Filtering with optional parameters
SELECT *
FROM users
WHERE
    (:email IS NULL OR email ILIKE '%' || :email || '%')
    AND (:role IS NULL OR role = :role)
    AND (:status IS NULL OR status = :status)
ORDER BY created_at DESC;
```

### Joins

```sql
-- Inner join
SELECT
    o.id AS order_id,
    o.total_amount,
    u.name AS user_name,
    u.email
FROM orders o
INNER JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed';

-- Left join with aggregation
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'completed'
GROUP BY u.id, u.name
ORDER BY total_spent DESC;

-- Lateral join for correlated subqueries
SELECT
    u.*,
    recent_orders.order_count,
    recent_orders.last_order_at
FROM users u
LEFT JOIN LATERAL (
    SELECT
        COUNT(*) AS order_count,
        MAX(created_at) AS last_order_at
    FROM orders
    WHERE user_id = u.id
    AND created_at > NOW() - INTERVAL '30 days'
) recent_orders ON true;
```

### Aggregations

```sql
-- Group by with filtering
SELECT
    DATE_TRUNC('day', created_at) AS day,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue
FROM orders
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE_TRUNC('day', created_at)
HAVING COUNT(*) > 10
ORDER BY day DESC;

-- Window functions
SELECT
    id,
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Running totals
SELECT
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;
```

### Common Table Expressions (CTEs)

```sql
-- Readable complex queries
WITH active_users AS (
    SELECT id, name, email
    FROM users
    WHERE status = 'active'
),
user_orders AS (
    SELECT
        user_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
)
SELECT
    au.name,
    au.email,
    COALESCE(uo.order_count, 0) AS order_count,
    COALESCE(uo.total_spent, 0) AS total_spent
FROM active_users au
LEFT JOIN user_orders uo ON au.id = uo.user_id
ORDER BY total_spent DESC;

-- Recursive CTE (for hierarchical data)
WITH RECURSIVE category_tree AS (
    -- Base case
    SELECT id, name, parent_id, 1 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT c.id, c.name, c.parent_id, ct.depth + 1
    FROM categories c
    INNER JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY depth, name;
```

## Indexing Strategy

```sql
-- B-tree (default) - equality and range queries
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_orders_created_at ON orders (created_at DESC);

-- Composite index - order matters!
-- Good for: WHERE user_id = ? AND status = ?
-- Good for: WHERE user_id = ?
-- NOT good for: WHERE status = ?
CREATE INDEX idx_orders_user_status ON orders (user_id, status);

-- Partial index - smaller, faster for specific queries
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';

-- GIN index for array/JSONB
CREATE INDEX idx_products_tags ON products USING GIN (tags);
CREATE INDEX idx_users_metadata ON users USING GIN (metadata jsonb_path_ops);

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);

-- Expression index
CREATE INDEX idx_users_email_lower ON users (LOWER(email));
```

## Transactions

```sql
-- Standard transaction
BEGIN;

INSERT INTO orders (user_id, total_amount)
VALUES ('user-123', 99.99)
RETURNING id INTO order_id;

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (order_id, 'prod-456', 2, 49.99);

UPDATE products
SET stock = stock - 2
WHERE id = 'prod-456' AND stock >= 2;

-- Check if update affected rows
IF NOT FOUND THEN
    ROLLBACK;
    RAISE EXCEPTION 'Insufficient stock';
END IF;

COMMIT;

-- With savepoints
BEGIN;

SAVEPOINT before_risky_operation;

-- Try something
UPDATE accounts SET balance = balance - 100 WHERE id = '123';

-- If it fails, rollback to savepoint
ROLLBACK TO SAVEPOINT before_risky_operation;

-- Continue with alternative
UPDATE accounts SET balance = balance - 50 WHERE id = '123';

COMMIT;
```

## Performance Tips

### Query Analysis
```sql
-- Analyze query plan
EXPLAIN ANALYZE
SELECT * FROM users
WHERE email = 'test@example.com';

-- Check index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE tablename = 'users';
```

### Avoid These Anti-Patterns
```sql
-- BAD: SELECT * (fetches unnecessary columns)
SELECT * FROM users;

-- GOOD: Select only needed columns
SELECT id, name, email FROM users;

-- BAD: N+1 queries
FOR user IN SELECT * FROM users LOOP
    SELECT * FROM orders WHERE user_id = user.id;
END LOOP;

-- GOOD: Single query with join
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- BAD: LIKE with leading wildcard (can't use index)
SELECT * FROM users WHERE email LIKE '%@example.com';

-- GOOD: Use pg_trgm extension for fuzzy search
CREATE INDEX idx_users_email_trgm ON users USING GIN (email gin_trgm_ops);

-- BAD: Implicit casting
SELECT * FROM users WHERE id = '123'; -- id is INTEGER

-- GOOD: Proper types
SELECT * FROM users WHERE id = 123;
```

## JSON Operations

```sql
-- JSONB column
ALTER TABLE users ADD COLUMN metadata JSONB DEFAULT '{}';

-- Query JSON
SELECT *
FROM users
WHERE metadata->>'subscription' = 'premium';

-- Query nested JSON
SELECT *
FROM users
WHERE metadata->'preferences'->>'theme' = 'dark';

-- Update JSON
UPDATE users
SET metadata = jsonb_set(metadata, '{preferences,notifications}', 'true')
WHERE id = '123';

-- Aggregate into JSON
SELECT jsonb_agg(jsonb_build_object(
    'id', id,
    'name', name
)) AS users
FROM users
WHERE status = 'active';
```
