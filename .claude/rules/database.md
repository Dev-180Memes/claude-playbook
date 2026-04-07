# Database

## Schema Design

- Use `snake_case` for table and column names: `user_profiles`, `created_at`
- Table names should be plural nouns: `users`, `orders`, `audit_logs`
- Every table must have a primary key — prefer surrogate keys (`id`) over composite natural keys unless the natural key is truly stable and unique
- Use `created_at` and `updated_at` timestamps on every table; populate them automatically at the database level
- Prefer explicit `NOT NULL` constraints — nullable columns should be intentional and documented
- Store monetary values as integers (cents) or `DECIMAL` with fixed precision — never `FLOAT`
- Store dates and times in UTC; handle timezone conversion in the application layer

## Naming

- Foreign key columns should be named `{referenced_table_singular}_id`: `user_id`, `order_id`
- Boolean columns should use affirmative names: `is_active`, `has_verified_email`
- Join/pivot tables should be named for both sides in alphabetical order: `post_tags`, `role_users`
- Index names should describe what they cover: `idx_users_email`, `idx_orders_user_id_created_at`

## Migrations

- Every schema change must go through a migration — never alter the production schema manually
- Migrations must be reversible where possible — write both `up` and `down` methods
- Never rename a column or table in a single migration if the app is live — use an expand/contract pattern:
  1. Add the new column
  2. Dual-write to both
  3. Backfill old data
  4. Switch reads to the new column
  5. Remove the old column in a later migration
- Do not perform long-running data backfills inside a migration transaction — run them as a separate background job
- Test migrations against a copy of production data before applying them

## Queries

- Select only the columns you need — avoid `SELECT *` in application code
- Always paginate queries that can return unbounded result sets
- Avoid N+1 queries — use eager loading, joins, or batch loading
- Wrap multi-step operations that must succeed or fail together in a transaction
- Use database-level constraints (foreign keys, unique, check) to enforce invariants — do not rely solely on application-level validation
- Avoid business logic in stored procedures or triggers; keep logic in the application layer where it can be tested and versioned

## Indexing

- Add an index on every foreign key column
- Add indexes for columns frequently used in `WHERE`, `ORDER BY`, or `JOIN` clauses
- Composite indexes should list the most selective column first
- Do not over-index — every index slows down writes and increases storage; remove unused indexes
- Create indexes `CONCURRENTLY` (or equivalent) on live tables to avoid locking

## Security

- Use parameterised queries for all user-supplied values — never concatenate input into SQL
- Grant the application database user only the permissions it needs (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) — not `DROP` or `ALTER`
- Never store plaintext passwords, tokens, or secrets in the database; hash or encrypt them first
- Audit access to tables containing PII or sensitive data

## General

- Document non-obvious column choices and constraints with a comment in the migration
- Keep the schema and seed data in version control alongside the application code
- Monitor slow query logs; set a query timeout appropriate to your workload
- Plan for growth — consider partitioning or archiving strategies before tables become unmanageably large
