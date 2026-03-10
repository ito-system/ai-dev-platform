# Skill: validate-database

## Responsibility
Validate that database migrations are consistent with the current schema and follow naming conventions.

## Input
- `migrations_path`: Path to the migrations directory
- `schema_path`: Path to the current schema definition file

## Checks
1. **Migration files exist**: Confirm at least one migration file exists for each new table or column referenced in the schema.
2. **Naming convention**: Confirm all migration files follow `YYYYMMDDHHMMSS_<description>.sql`.
3. **Down migration**: Confirm every `up` migration has a corresponding `down` migration.
4. **No duplicate timestamps**: Confirm no two migration files share the same timestamp prefix.
5. **Foreign keys valid**: Confirm all foreign keys reference tables that exist in the schema.

## Output
- Pass/Fail for each check
- List of violations
- Overall status: PASS or FAIL

## Notes
- Read `shared-context/database-design-guidelines.md` before running checks.
