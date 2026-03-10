# Playbook: add-feature

## Purpose
Add a fully specified, tested, and validated new feature to an existing SaaS product without breaking existing functionality — from user story through migration, backend, frontend, tests, validation, and commit.

## Role
Read `prompts/system/architect-role.md` before starting. Pay special attention to the impact-on-existing-features analysis in the specification phase.

## Inputs

| Parameter | Required | Description |
|---|---|---|
| `product_name` | required | Display name of the SaaS product (e.g., `"OnboardIQ"`) |
| `feature_name` | required | Name of the feature to add (e.g., `"Email Notification Digest"`) |
| `user_story` | required | `"As a <role>, I want <action> so that <benefit>."` |
| `repo_path` | required | Absolute path to the local repository root |
| `priority` | optional, default `"standard"` | `"critical"` \| `"standard"` \| `"low"` |

## Preconditions

Before starting, verify all of the following are true:

- [ ] The repository exists at `repo_path`.
- [ ] The user story follows the exact format: `"As a <role>, I want <action> so that <benefit>."`
- [ ] There are no uncommitted changes in the working directory (`git status` is clean).
- [ ] The project builds without errors (`go build ./...` passes).
- [ ] The existing test suite passes (`go test ./...` passes).
- [ ] The database is in a consistent migration state (no pending migrations).

**If any precondition is not met, stop. Resolve before starting this playbook. Do not add a feature on top of a broken baseline.**

## Error Handling Policy

- **Specification phase failures** (Phase 1): Stop and report. No code has been written. Fix inputs or the generated spec, then re-run the phase.
- **Breaking change detected in spec**: Stop immediately when a `[BREAKING]` flag appears in the feature spec. Do not proceed until the migration path is defined and approved.
- **Implementation phase failures** (Phase 2): Stop at the failing step. Report the error. Roll back files generated in the current step only. Files from prior completed steps are safe.
- **Validation failures** (Phase 3): Stop. Identify the failing artifact. Return to the responsible implementation step, fix, then re-run the **full validation phase** (all four validators). Do not run partial validation.
- **Commit failure** (Phase 4): Implementation is complete and validated. Fix the commit issue without touching implementation files. Retry the commit step only.

---

## Phase 1: Specification

**Goal:** Produce a complete feature specification that documents all new endpoints, schema changes, UI components, edge cases, and impact on existing features — before writing any code.

### Step 1 — Read Shared Context
Read the following documents in order:
1. `shared-context/go-clean-architecture.md`
2. `shared-context/api-design-guidelines.md`
3. `shared-context/database-design-guidelines.md`
4. `shared-context/react-frontend-guidelines.md`

---

### Step 2 — Collect Existing Project State
Before generating the feature spec, collect the current state of the codebase:

1. List all domain entities defined in `internal/domain/` → capture as `existing_entities`
2. List all registered API routes in `routes/routes.go` → capture as `existing_endpoints`
3. List all existing migration files in `database/migrations/` → capture last migration number as `last_migration_number`

This information is required inputs to the next step. Do not skip this collection.

---

### Step 3 — Generate Feature Specification
Execute skill: `skills/tob-saas/generate-feature.md`

Input:
- `product_name` → from playbook input
- `feature_name` → from playbook input
- `user_story` → from playbook input
- `existing_entities` → collected in Step 2
- `existing_endpoints` → collected in Step 2
- `priority` → from playbook input

Output to capture:
- `feature_spec` — complete specification document
- `acceptance_criteria` — used in Step 13 (validate-feature)
- `api_changes` — new and modified endpoints; used in Step 8
- `data_model_changes` — new entities, modified entities, migration list; used in Step 7
- `ui_components` — new components; used in Step 9
- `edge_cases` — used in Step 13 (validate-feature)
- `impact_assessment` — checked in next step
- `breaking_changes` — checked in next step

**Error handling:** If the skill's Validation checklist has any unchecked item, fix the spec before proceeding. An incomplete spec will produce an incorrect implementation.

---

### Step 4 — Review Breaking Changes and Impact
Review the `impact_assessment` and `breaking_changes` sections of the spec from Step 3.

**If breaking changes are present `[BREAKING]`:**
- Stop.
- Report the breaking change and its documented migration path to the operator.
- Do not proceed until the migration path is explicitly approved.
- Record the approval decision before continuing.

**If no breaking changes:** Confirm the impact assessment explicitly states "No impact on existing features" or lists the affected components. If neither is present, return to Step 3 and complete the impact assessment.

---

### Step 5 — Verify No API Endpoint Conflicts
Cross-check every new endpoint in `api_changes` against `existing_endpoints` from Step 2.

For each new endpoint (method + path pair):
- If the exact pair already exists: **stop**. The spec has a conflict — return to Step 3 and resolve.
- If the path exists with a different method: acceptable only if the methods are semantically distinct (e.g., GET and POST on the same collection path). Document the rationale.

**Output:** Confirmed conflict-free endpoint list.

---

## Phase 2: Implementation

**Goal:** Generate all implementation artifacts in dependency order.

**Implementation order is fixed. Schema first, then backend, then frontend, then tests.**

---

### Step 6 — Determine Migration Requirement
Inspect `data_model_changes` from Step 3.

- If `data_model_changes.required_migrations` is non-empty → proceed to Step 7.
- If `data_model_changes.required_migrations` is empty → skip Step 7 and proceed directly to Step 8.

Record this decision in the session log.

---

### Step 7 — Generate Database Migration (conditional)
*Skip this step if determined non-required in Step 6.*

Execute skill: `skills/development/generate-migration.md`

Input:
- `entities` → new entities from `data_model_changes`
- `modified_entities` → modified entities from `data_model_changes`
- `migration_number` → `last_migration_number + 1` (from Step 2)
- `tenant_scoped_entities` → all entities where `tenant-scoped: yes`

Expected output:
- SQL migration file(s) in `database/migrations/` numbered sequentially
- For modified entities: `ALTER TABLE` statements only — no `DROP COLUMN` unless explicitly specified in the spec

**Rollback on failure:** Delete the generated migration file(s). Re-run this step. Do not proceed with invalid SQL.

**Error handling:** If a new entity name conflicts with an existing table, stop. Return to Step 3 and resolve the naming conflict in the spec.

---

### Step 8 — Generate Backend API Changes
Execute skill: `skills/development/generate-backend-api.md`

Input:
- `resource` → primary resource name for the feature
- `operations` → operations from `api_changes.new_endpoints`

For **modified** endpoints (listed in `api_changes.modified_endpoints`):
- Read the existing handler, use-case, or repository file.
- Add only the delta defined in the spec. Do not remove existing functionality.
- Verify the modification does not change existing function signatures in a breaking way.

Expected output:
- New files following `go-clean-architecture.md` naming conventions
- Modified files: only the delta applied, existing code preserved

**Rollback on failure:** Delete new files generated in this step. Restore modified files from git (`git checkout -- <file>`). Re-run from this step.

**Error handling:** If a modification would change an existing function signature, stop. Flag as a breaking change and return to Phase 1 (Step 3) to document the migration path.

---

### Step 9 — Generate Frontend UI
Execute skill: `skills/development/generate-frontend-ui.md`

Input:
- `components` → `ui_components` from Step 3
- `api_contract` → `api_changes.new_endpoints` from Step 3

For new components only. Do not regenerate existing components unless they are explicitly listed in `impact_assessment` as requiring modification.

**Rollback on failure:** Delete new component files. Restore modified files from git. Re-run this step only.

---

### Step 10 — Generate Tests
Execute skill: `skills/development/generate-tests.md`

Input:
- `use_case_files` → new or modified use-case file(s) from Step 8
- `handler_files` → new or modified handler file(s) from Step 8
- `frontend_components` → new component names from Step 9
- `edge_cases` → from Step 3 (to generate edge-case test scenarios)

Expected output:
- New test files for new use-cases and handlers
- Updated test files for modified use-cases and handlers (add test cases; do not remove existing ones)
- Edge-case test scenarios for each edge case listed in the spec

**Error handling:** If modifying an existing test file, read the full file first. Add new test cases only. Do not overwrite or delete existing test cases.

---

## Phase 3: Validation

**Goal:** Confirm the new feature is correct and has not broken any existing functionality.

**Run all four validators. Stop on the first FAIL. Do not run partial validation.**

---

### Step 11 — Validate Database
Execute skill: `skills/validation/validate-database.md`

Additional checks (beyond baseline):
- New migration number is `last_migration_number + 1` (no gaps, no duplicates)
- All new tenant-scoped tables include `organization_id` with FK
- No `DROP COLUMN` or `DROP TABLE` unless explicitly documented in the spec as an approved breaking change

**On FAIL:** Fix the migration from Step 7. Re-run Steps 11–14 (full validation phase).

---

### Step 12 — Validate Backend
Execute skill: `skills/validation/validate-backend.md`

Additional checks for modified files:
- Existing function signatures are unchanged (unless a `[BREAKING]` change was approved in Phase 1)
- New use-case methods follow existing patterns in the file
- No new global variables or package-level state introduced

**On FAIL:** Fix the identified file from Step 8. Re-run Steps 11–14.

---

### Step 13 — Validate Frontend
Execute skill: `skills/validation/validate-frontend.md`

Additional checks:
- Existing components not listed in `impact_assessment` are unmodified
- New components follow the same patterns as existing components in the project

**On FAIL:** Fix the identified component from Step 9. Re-run Steps 11–14.

---

### Step 14 — Validate Feature
Execute skill: `skills/validation/validate-feature.md`

Input:
- `acceptance_criteria` → from Step 3
- `edge_cases` → from Step 3

Checks:
- All acceptance criteria have corresponding implementations
- All edge cases from the spec are handled (in code or documented as explicit test cases)
- No acceptance criterion is partially implemented

**On FAIL:** Identify the unmet criterion or edge case. Determine the responsible layer. Fix in the appropriate Phase 2 step, then re-run Steps 11–14.

---

## Phase 4: Commit

**Only execute after all Phase 3 validations return PASS.**

---

### Step 15 — Verify Baseline Still Passes
Before committing, run a final check:

- Confirm the existing test suite still passes with the new code in place.
- If tests that existed before this playbook now fail: **stop**. A regression has been introduced. Return to the responsible implementation step, fix, and re-run validation.

---

### Step 16 — Commit Changes
Execute skill: `skills/github/commit-changes.md`

Input:
- `message` → `"feat: add <feature_name>"`

Include in the commit:
- New and modified migration files (Step 7, if applicable)
- New and modified backend files (Step 8)
- New and modified frontend files (Step 9)
- New and modified test files (Step 10)

Do not include:
- `.env` files
- Compiled artifacts
- Files unrelated to this feature

**Error handling:** If a pre-commit hook fails, read the output, fix the reported issue, re-stage, and retry. Never use `--no-verify`.

---

## Output

```
Feature Addition Summary
────────────────────────
Product:          <product_name>
Feature:          <feature_name>
Priority:         <priority>
Commit:           <commit_hash>

Migration:        <yes — migration N_<name>.sql | no — no schema changes>
Breaking Changes: <none | [BREAKING] <description>>

New Files:
  <list of created files>

Modified Files:
  <list of modified files>

Validation Results:
  Database:  PASS | SKIPPED (no migration)
  Backend:   PASS
  Frontend:  PASS
  Feature:   PASS

Acceptance Criteria:
  [x] <criterion 1>
  [x] <criterion 2>
  ...

Edge Cases Handled:
  [x] <edge case 1>
  [x] <edge case 2>
  ...
```
