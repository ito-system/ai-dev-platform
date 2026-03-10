# Playbook: generate-core-feature

## Purpose
Specify and implement the single most critical feature of an existing SaaS product — the feature that most directly delivers the product's value proposition — including migration, full-stack implementation, tests, and validation.

## Role
Read `prompts/system/architect-role.md` before starting. Apply architect-level judgment when resolving ambiguity in the feature spec.

## Inputs

| Parameter | Required | Description |
|---|---|---|
| `product_name` | required | Display name of the SaaS product (e.g., `"OnboardIQ"`) |
| `value_proposition` | required | One-sentence VP (e.g., from `create-saas` playbook output or product docs) |
| `feature_name` | required | Name of the core feature to implement (e.g., `"Automated Task Assignment"`) |
| `target_user` | required | Primary user persona (e.g., `"HR manager at a 50-person company"`) |
| `repo_path` | required | Absolute path to the local repository root |
| `existing_entities` | optional | Comma-separated list of domain entities already defined (e.g., `"User,Organization"`) |

## Preconditions

Before starting, verify all of the following are true:

- [ ] The repository exists at `repo_path`.
- [ ] A Go module is initialized (`go.mod` exists).
- [ ] The database connection is configured (`.env` or equivalent config is present).
- [ ] Auth middleware is already implemented (protected endpoints require authenticated context).
- [ ] No uncommitted changes exist in the working directory (`git status` is clean).

**If any precondition is not met, stop. Resolve the precondition before starting this playbook.**

## Error Handling Policy

- **Specification phase failures** (Phase 1): Stop and report. Nothing has been written to the codebase. Revise inputs and re-run Phase 1.
- **Implementation phase failures** (Phase 2): Stop at the failing step. Do not proceed. Report the error with context. Roll back any generated files from the current step before retrying. Files from successfully completed prior steps are safe and do not need to be rolled back.
- **Validation failures** (Phase 3): Stop. Identify the failing validation check. Return to the step in Phase 2 that produced the failing artifact. Fix and re-run the full validation phase.
- **Commit failure** (Phase 4): The implementation is complete and validated. Fix the commit issue (e.g., pre-commit hook error) without modifying implementation files. Re-run the commit step only.

---

## Phase 1: Specification

**Goal:** Produce a complete, unambiguous feature specification that all implementation steps can follow without further clarification.

### Step 1 — Read Shared Context
Read the following documents in order:
1. `shared-context/go-clean-architecture.md` — understand layer responsibilities and dependency rules
2. `shared-context/api-design-guidelines.md` — understand endpoint, response, and error conventions
3. `shared-context/database-design-guidelines.md` — understand schema and migration conventions

Do not proceed to Step 2 until all three documents have been read.

---

### Step 2 — Generate Core Feature Specification
Execute skill: `skills/tob-saas/generate-core-feature.md`

Input:
- `product_name` → from playbook input
- `value_proposition` → from playbook input
- `feature_name` → from playbook input
- `target_user` → from playbook input
- `data_entities` → `existing_entities` from playbook input (empty list if not provided)

Output to capture:
- `feature_spec` — complete specification document
- `user_story` — verified format before proceeding
- `acceptance_criteria` — checklist; used in Step 11 (validate-feature)
- `api_contract` — endpoint definitions; used in Step 6 (generate-backend-api)
- `data_model` — entity definitions; used in Step 5 (generate-migration)
- `ui_components` — component list; used in Step 7 (generate-frontend-ui)
- `prerequisites` — list verified in next step

**Error handling:** If the skill's Validation checklist has any unchecked item, fix the spec output before proceeding. Do not start implementation from an incomplete spec.

---

### Step 3 — Verify Prerequisites
Review the `prerequisites` section of the feature spec from Step 2.

For each listed prerequisite:
- Check whether it already exists in the codebase at `repo_path`.
- If it exists: mark as satisfied.
- If it does not exist: **stop**. Document the missing prerequisite and resolve it before continuing.

Common prerequisites to check:
- Auth middleware (`internal/middleware/auth.go`)
- Tenant middleware (`internal/middleware/tenant.go`)
- Organization entity and migration
- User entity and migration

**Output:** A signed-off prerequisite checklist. All items must be satisfied before Phase 2 begins.

---

## Phase 2: Implementation

**Goal:** Generate all implementation artifacts in dependency order — schema first, then backend, then frontend, then tests.

**Implementation order is fixed. Do not reorder steps.**

**Read before implementing (if not already read in Phase 1):**
- `shared-context/go-clean-architecture.md`
- `shared-context/api-design-guidelines.md`
- `shared-context/database-design-guidelines.md`
- `shared-context/react-frontend-guidelines.md`

---

### Step 4 — Generate Database Migration
Execute skill: `skills/development/generate-migration.md`

Input:
- `entities` → `data_model` from Step 2
- `tenant_scoped_entities` → all entities where `tenant-scoped: yes` in the data model

Expected output:
- SQL migration file(s) in `database/migrations/`
- Migration numbered sequentially after the last existing migration

**Rollback on failure:** If migration file generation fails or produces invalid SQL, delete the generated file(s) and re-run this step. Do not proceed with a broken migration.

**Error handling:** If an entity in `data_model` conflicts with an existing table name (check existing migrations), stop. Resolve the naming conflict in the spec (Step 2) and re-run from Step 2.

---

### Step 5 — Generate Backend API
Execute skill: `skills/development/generate-backend-api.md`

Input:
- `resource` → primary resource name from the feature spec (e.g., `"task"`)
- `operations` → operations defined in `api_contract` from Step 2

Expected output files (following `go-clean-architecture.md` naming):
- `internal/domain/<resource>.go` — domain entity
- `internal/usecase/<resource>_repository.go` — repository interface
- `internal/usecase/<resource>_usecase.go` — use-case implementation
- `internal/repository/<resource>_repository.go` — repository implementation
- `internal/handler/<resource>_handler.go` — HTTP handler
- `internal/dto/<resource>_dto.go` — request/response DTOs
- Route registration snippet for `routes/routes.go`

**Rollback on failure:** If any layer fails to generate, delete all files generated in this step (check the expected output list) and re-run from this step. Do not leave partial layer implementations.

**Error handling:** If the generated handler does not follow the API response envelope from `api-design-guidelines.md`, fix before proceeding to Step 6.

---

### Step 6 — Generate Frontend UI
Execute skill: `skills/development/generate-frontend-ui.md`

Input:
- `components` → `ui_components` from Step 2
- `api_contract` → endpoint list from Step 2 (for API integration)

Expected output:
- React component files for each item in `ui_components`
- API client integration (fetch/axios calls matching the generated endpoints)
- Loading state and error state handling for each component

**Rollback on failure:** Delete all frontend files generated in this step. Re-run from this step. Steps 4 and 5 output is safe.

**Error handling:** If a component requires an API endpoint not in the `api_contract`, stop. Update the spec (return to Step 2) and re-generate the backend (Step 5) first.

---

### Step 7 — Generate Tests
Execute skill: `skills/development/generate-tests.md`

Input:
- `use_case_files` → use-case file(s) generated in Step 5
- `handler_files` → handler file(s) generated in Step 5
- `frontend_components` → component names from Step 6

Expected output:
- `internal/usecase/<resource>_usecase_test.go` — use-case unit tests with mocked repository
- `internal/handler/<resource>_handler_test.go` — handler tests with `httptest` and mocked use-case
- Frontend component test files

**Rollback on failure:** Delete generated test files. Re-run this step only.

**Error handling:** If the test generator cannot produce tests because interfaces are unclear, fix the use-case or handler implementation (Step 5) first, then re-run this step.

---

## Phase 3: Validation

**Goal:** Confirm all generated artifacts are correct and complete before committing.

**All validation skills must return PASS. Stop on the first FAIL.**

---

### Step 8 — Validate Database
Execute skill: `skills/validation/validate-database.md`

Checks:
- Migration SQL is syntactically valid
- All tenant-scoped tables include `organization_id` column with FK to `organizations`
- FK columns have indexes
- `created_at` and `updated_at` columns are present on all entities
- Migration number is sequential (no gaps)

**On FAIL:** Fix the migration file from Step 4. Re-run Step 8 only.

---

### Step 9 — Validate Backend
Execute skill: `skills/validation/validate-backend.md`

Checks:
- No reverse layer dependencies (handler → usecase → repository → domain)
- Repository interface is defined in the `usecase` package
- All handler functions use `{ "data": ... }` or `{ "error": ... }` response envelopes
- Domain errors are mapped to correct HTTP status codes in the handler
- All use-case and repository functions accept `context.Context` as first parameter
- `organization_id` is applied in every repository query that returns tenant-scoped data

**On FAIL:** Fix the identified file from Phase 2 (Step 5). Re-run Step 9 only. If the fix changes function signatures, re-run Steps 9 and 10 (tests may need updating).

---

### Step 10 — Validate Frontend
Execute skill: `skills/validation/validate-frontend.md`

Checks:
- Each component renders a loading state
- Each component renders an error state
- API calls use the correct endpoint paths from the `api_contract`
- No hardcoded API URLs (use environment variable or config constant)

**On FAIL:** Fix the identified component from Step 6. Re-run Step 10 only.

---

### Step 11 — Validate Feature
Execute skill: `skills/validation/validate-feature.md`

Input:
- `acceptance_criteria` → from Step 2

Checks:
- Each acceptance criterion from the spec has a corresponding implementation
- Each criterion is verifiable via the generated tests or manual check description

**On FAIL:** Identify the unmet criterion. Determine whether the fix is in the backend (Step 5), frontend (Step 6), or tests (Step 7). Fix, then re-run the **full validation phase** (Steps 8–11).

---

## Phase 4: Commit

**Goal:** Record the validated implementation as a single atomic commit.

**Only execute after all Phase 3 validations return PASS.**

---

### Step 12 — Commit Changes
Execute skill: `skills/github/commit-changes.md`

Input:
- `message` → `"feat: implement core feature — <feature_name>"`

Include in the commit:
- All migration files from Step 4
- All backend files from Step 5
- All frontend files from Step 6
- All test files from Step 7
- Route registration changes

Do not include:
- `.env` files
- Local development config overrides
- Binary files or compiled artifacts

**Error handling:** If a pre-commit hook fails, read the hook's output, fix the reported issue, re-stage, and re-run the commit. Never use `--no-verify`.

---

## Output

```
Core Feature Implementation Summary
─────────────────────────────────────
Product:          <product_name>
Feature:          <feature_name>
Commit:           <commit_hash>

Generated Files:
  Migration:  database/migrations/<N>_create_<resource>.sql
  Domain:     internal/domain/<resource>.go
  Use-case:   internal/usecase/<resource>_usecase.go
  Repository: internal/repository/<resource>_repository.go
  Handler:    internal/handler/<resource>_handler.go
  DTO:        internal/dto/<resource>_dto.go
  Tests:      internal/usecase/<resource>_usecase_test.go
              internal/handler/<resource>_handler_test.go
  Frontend:   <component files>

Validation Results:
  Database:  PASS
  Backend:   PASS
  Frontend:  PASS
  Feature:   PASS

Acceptance Criteria:
  [x] <criterion 1>
  [x] <criterion 2>
  ...
```
