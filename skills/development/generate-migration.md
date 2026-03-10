# Skill: generate-migration

## Responsibility
Generate a database migration file for a given schema change.

## Input
- `table_name`: Name of the table being created or altered
- `changes`: Description of the schema changes (e.g., "add email column as unique not null")
- `direction`: `up` only, or both `up` and `down`

## Steps
1. Read `shared-context/database-design-guidelines.md`.
2. Generate the SQL migration for the `up` direction.
3. If requested, generate the `down` rollback migration.
4. Name the file with a timestamp prefix: `YYYYMMDDHHMMSS_<description>.sql`.

## Output
- Migration file content (up + optional down)
- Suggested filename

## Notes
- Always include a rollback (`down`) for destructive changes.
- Foreign keys must reference existing tables.
- Use snake_case for column names.
