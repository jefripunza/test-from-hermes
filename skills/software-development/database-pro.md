---
name: database-pro
description: "Database expert: PostgreSQL, MySQL, Redis. Schema design, index optimization, query optimization, transaction design, migration planning. Prevent N+1."
metadata:
  hermes:
    tags: [database, postgresql, mysql, redis, sql, query-optimization, indexing]
---

# Database Pro — Database Expert

PostgreSQL, MySQL, Redis. Schema design, optimization, transactions, migrations.

## Expertise

- **PostgreSQL** — advanced features, JSONB, full-text search, partitioning, PL/pgSQL
- **MySQL** — InnoDB, replication, performance schema
- **Redis** — caching, pub/sub, streams, sorted sets, rate limiting
- Schema design — normalization, denormalization tradeoffs
- Index optimization — B-tree, GiST, GIN, BRIN, partial indexes, covering indexes
- Query optimization — EXPLAIN ANALYZE, slow query log
- Transaction design — isolation levels, locking, deadlock prevention
- Migration planning — zero-downtime migrations, backward compatibility
- N+1 prevention — eager loading, batch queries
- Connection pooling — PgBouncer, connection limits

## Requirements

- **Prevent N+1 queries** — use JOIN, batch loading, dataloader pattern
- **Optimize indexes** — EXPLAIN ANALYZE every query, check seq scans
- **Consider scalability** — sharding, read replicas, partitioning strategy
- **Explain tradeoffs** — normalization vs performance, consistency vs availability

## Index Strategy

```sql
-- B-tree (default) — equality + range queries
CREATE INDEX idx_users_email ON users(email);

-- Composite — order matters (equality first, then range)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial — index subset of rows
CREATE INDEX idx_active_orders ON orders(status) WHERE status = 'active';

-- Covering — index-only scan
CREATE INDEX idx_users_email_cover ON users(email) INCLUDE (name, avatar_url);

-- GIN — JSONB, full-text, arrays
CREATE INDEX idx_metadata ON products USING GIN(metadata);

-- BRIN — large tables with natural ordering
CREATE INDEX idx_created_at ON logs USING BRIN(created_at);
```

## Migration Pattern (Zero-Downtime)

```sql
-- Phase 1: Add column (nullable, no default)
ALTER TABLE users ADD COLUMN email_verified BOOLEAN;

-- Phase 2: Backfill in batches
UPDATE users SET email_verified = TRUE WHERE email_verified IS NULL LIMIT 1000;

-- Phase 3: Add NOT NULL + default
ALTER TABLE users ALTER COLUMN email_verified SET NOT NULL;
ALTER TABLE users ALTER COLUMN email_verified SET DEFAULT FALSE;
```

## Query Analysis

```sql
EXPLAIN ANALYZE SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;
-- Look for: seq scan, large loops, temp files, high cost
```

## Redis Patterns

```typescript
// Cache-aside
async function getUser(id: string) {
  const cached = await redis.get(`user:${id}`)
  if (cached) return JSON.parse(cached)
  const user = await db.findUser(id)
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600)
  return user
}
```
