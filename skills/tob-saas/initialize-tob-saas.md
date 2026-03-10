# Skill: initialize-tob-saas

## Responsibility
Prepare a newly cloned SaaS repository for immediate development by setting up the environment file, starting Docker containers, running database migrations, and applying initial seed data — then verifying the baseline build and test suite pass.

---

## Input

- `repo_path` (required): Absolute path to the local repository root
- `env_file` (optional, default: `<repo_path>/.env.example`): Path to the environment template file to copy from
- `run_docker` (optional, default: `true`): Start Docker containers via `docker compose up -d`
- `run_migrations` (optional, default: `true`): Run pending database migrations inside the backend container
- `run_seed` (optional, default: `true`): Apply initial seed data inside the backend container

---

## Steps

### Step 1 — Copy Environment File

1. Determine the source path:
   - If `env_file` is provided, use it as-is.
   - Otherwise, use `<repo_path>/.env.example`.
2. Confirm the source file exists. If it does not exist, **stop** and report:
   `ERROR: env template not found at <source_path>. Create .env.example before running this skill.`
3. Check whether `<repo_path>/.env` already exists:
   - If it exists: **do not overwrite**. Log `SKIP — .env already exists` and proceed to Step 2.
   - If it does not exist: copy source → `<repo_path>/.env`.
4. Confirm `.env` is listed in `.gitignore`. If it is not:
   - **Stop** and report: `ERROR: .env is not in .gitignore. Add it before continuing to prevent accidental credential commits.`
5. Confirm the following mandatory variables are defined (non-empty) in `.env`:
   - `DATABASE_URL`
   - `APP_ENV`
   - `APP_PORT`
   If any mandatory variable is missing or empty, **stop** and report each missing variable by name.

**Idempotency:** If `.env` already exists with all mandatory variables set, this step is a no-op.

---

### Step 2 — Start Docker Environment

*Skip this step if `run_docker` is `false`.*

1. Run: `docker compose up -d` in `repo_path`.
2. Wait for containers to reach a healthy state (poll `docker compose ps` up to 30 seconds).
3. Confirm all defined services have status `running` or `healthy`.
   - If any service has status `exited` or `restarting`: **stop**.
   - Fetch and report logs for the failing service: `docker compose logs <service_name> --tail=50`.
   - Do not proceed until all containers are healthy.

Expected running services (verify at minimum):
- `backend` — Go API server
- `db` — PostgreSQL database
- `frontend` — React dev server (if defined in `docker-compose.yml`)

**Idempotency:** If containers are already running, `docker compose up -d` is a no-op. Re-running this step is safe.

---

### Step 3 — Run Database Migrations

*Skip this step if `run_migrations` is `false`.*

1. Check for pending migrations:
   ```bash
   docker compose exec backend migrate \
     -path /app/database/migrations \
     -database "$DATABASE_URL" \
     version
   ```
2. If no pending migrations exist, log `SKIP — database is already up to date` and continue.
3. If pending migrations exist, apply them:
   ```bash
   docker compose exec backend migrate \
     -path /app/database/migrations \
     -database "$DATABASE_URL" \
     up
   ```
4. After execution, verify the migration ran successfully:
   - Re-run the `version` command and confirm the version has advanced.
   - Confirm exit code is `0`.
5. If migration fails:
   - **Stop** immediately. Do not proceed to seed step.
   - Report the failing migration file name and error message.
   - Do not attempt a rollback automatically — migration rollback requires human review.

**Idempotency:** Running this step when no pending migrations exist is a no-op. Re-running after a successful migration is safe.

---

### Step 4 — Run Initial Seed Data

*Skip this step if `run_seed` is `false`.*

1. Check whether seed data already exists (idempotency guard):
   ```bash
   docker compose exec backend go run backend/database/seeds/seed.go --check
   ```
   - If the seed script reports data already present: log `SKIP — seed data already exists` and continue.
   - If the seed script is not idempotent (no `--check` flag): query the `organizations` table directly:
     ```bash
     docker compose exec db psql "$DATABASE_URL" -c "SELECT COUNT(*) FROM organizations;"
     ```
     If count > 0: log `SKIP — seed data already exists` and continue.
2. If no seed data exists, apply it:
   ```bash
   docker compose exec backend go run backend/database/seeds/seed.go
   ```
3. After execution, verify at least one record exists in each core table:
   - `users` — at least 1 row
   - `organizations` — at least 1 row
4. If seeding fails:
   - **Stop** and report the error output.
   - Do not roll back migrations — seed failure is isolated.

**Idempotency:** The idempotency guard in Step 4.1 ensures seed data is never duplicated. Re-running this step when seed data exists is safe.

---

### Step 5 — Verify Baseline

1. **Backend build check:**
   ```bash
   docker compose exec backend go build ./...
   ```
   - If the build fails: **stop**. Report the compiler errors in full.
   - Do not proceed to tests if the build fails.

2. **Backend test suite:**
   ```bash
   docker compose exec backend go test ./...
   ```
   - If any test fails: **stop**. Report the failing test names and error messages.
   - A failing baseline test suite means the repository was already broken before this skill ran. Document this and do not suppress test failures.

3. **Frontend check (if applicable):**
   - If a `frontend/` directory exists in `repo_path`:
     ```bash
     docker compose exec frontend npm run build
     ```
   - If build fails: **stop** and report errors.

**Note:** This step verifies the repository baseline — it does not test any feature implemented after initialization. All tests should pass on a freshly initialized repository.

---

## Output

```
SaaS Initialization Report
───────────────────────────
Repository:       <repo_path>
Timestamp:        <ISO 8601 datetime>

Environment:
  .env:           CREATED | SKIPPED (already exists)
  Mandatory vars: ALL PRESENT | MISSING: <var1>, <var2>

Docker Containers:
  backend:        running | ERROR — <reason>
  db:             running | ERROR — <reason>
  frontend:       running | SKIPPED (not defined) | ERROR — <reason>

Migrations:
  Status:         APPLIED (<N> migrations) | SKIPPED (up to date) | ERROR — <migration_file>
  DB version:     <migration version number>

Seed Data:
  Status:         APPLIED | SKIPPED (already exists) | ERROR — <reason>
  Records:        users=<N>, organizations=<N>

Baseline Verification:
  Backend build:  PASS | FAIL — <error summary>
  Backend tests:  PASS (<N> tests) | FAIL — <failing test list>
  Frontend build: PASS | SKIPPED | FAIL — <error summary>

Overall Status:   READY | FAILED
```

---

## Validation

- [ ] `.env` file exists at `<repo_path>/.env` and is not tracked by git.
- [ ] All mandatory environment variables (`DATABASE_URL`, `APP_ENV`, `APP_PORT`) are defined and non-empty in `.env`.
- [ ] All Docker services defined in `docker-compose.yml` have status `running` or `healthy` (if `run_docker: true`).
- [ ] Migration version is up to date — no pending migrations remain (if `run_migrations: true`).
- [ ] At least 1 row exists in `users` and `organizations` tables (if `run_seed: true`).
- [ ] `go build ./...` exits with code `0`.
- [ ] `go test ./...` exits with code `0` and no test failures.
- [ ] Overall Status in the output report is `READY`.

---

## Notes

- This skill is executed **once**, immediately after cloning the repository and running `skills/project/replace-project-name.md`.
- Seed data must consist of **minimal, non-sensitive dummy records only**. Never seed production credentials, real user emails, or payment data.
- If the repository uses a migration tool other than `golang-migrate` (e.g., `goose`, `atlas`), adapt the migration commands in Steps 3.1–3.3 accordingly. Document the tool in the project's `README.md`.
- Do not run this skill against a production environment. It is intended for local development setup only.
- The `--check` flag for the seed script is a convention. If the seed script does not support it, use the direct table count query as the idempotency guard.
