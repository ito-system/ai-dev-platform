# Database Design Guidelines

This document defines database schema and migration conventions.
All migration generation skills must follow this guide.

---

## General conventions

- Use PostgreSQL.
- All table names: plural snake_case (`users`, `subscription_plans`).
- All column names: snake_case.
- Every table must have: `id`, `created_at`, `updated_at`.
- Primary keys: UUID (`uuid` type), not auto-increment integer.

---

## Standard columns

```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

---

## Naming conventions

| Object | Convention | Example |
|---|---|---|
| Table | plural snake_case | `user_profiles` |
| Column | snake_case | `first_name` |
| Index | `idx_<table>_<column>` | `idx_users_email` |
| Foreign key | `fk_<table>_<ref_table>` | `fk_posts_users` |
| Unique constraint | `uq_<table>_<column>` | `uq_users_email` |

---

## Migration conventions

- File naming: `YYYYMMDDHHMMSS_<description>.sql`
- Every migration must have `-- migrate:up` and `-- migrate:down` sections.
- Never DROP columns in `up` without a corresponding restore in `down`.
- Never use irreversible operations without explicit justification.

```sql
-- migrate:up
ALTER TABLE users ADD COLUMN stripe_customer_id TEXT;

-- migrate:down
ALTER TABLE users DROP COLUMN stripe_customer_id;
```

---

## Soft deletes

Use `deleted_at TIMESTAMPTZ` for soft deletes. Do not use hard deletes for user-facing data.

---

## Indexes

- Index all foreign key columns.
- Index columns used in WHERE clauses for large tables.
- Use partial indexes for boolean flags (e.g., `WHERE is_active = true`).
