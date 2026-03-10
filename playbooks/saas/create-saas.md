# Playbook: create-saas

## Purpose
Generate a complete, production-ready SaaS application scaffold — from product concept through brand identity, core feature specification, full-stack implementation, validation, and repository setup — in a single automated workflow.

## Role
Read `prompts/system/architect-role.md` before starting. Apply architect-level judgment throughout all phases.

## Inputs

| Parameter | Required | Description |
|---|---|---|
| `market` | required | Target market or industry segment (e.g., `"HR teams at SMBs"`) |
| `problem_domain` | required | Area of pain to solve (e.g., `"employee onboarding"`) |
| `constraints` | optional | Known constraints (e.g., `"no hardware"`, `"1–2 engineer team"`) |
| `product_name` | optional | Pre-defined name. If omitted, generated in Phase 1. |
| `tone_preference` | optional | Brand tone: `"professional"` \| `"friendly"` \| `"bold"` \| `"minimal"` |

## Error Handling Policy

- **Ideation phase failures** (Phases 1–2): Stop and report. No code has been written. Re-run with revised inputs.
- **Implementation phase failures** (Phase 3): Stop immediately. Do not proceed to validation. Report the failing step and its output. Do not commit partial work.
- **Validation failures** (Phase 4): Stop. Do not proceed to repository setup. Fix the reported issue by re-running the failing implementation step, then re-run the full validation phase.
- **Repository setup failures** (Phase 5): Implementation is complete and validated. Report the failure and retry the specific github skill that failed. Do not re-run implementation.

---

## Phase 1: Ideation

**Goal:** Produce a validated product concept, name, and brand identity.

### Step 1 — Generate SaaS Idea
Execute skill: `skills/saas/generate-saas-idea.md`

Input:
- `market` → from playbook input
- `problem_domain` → from playbook input
- `constraints` → from playbook input (pass empty if not provided)

Output to capture:
- `value_proposition` — used in Steps 2, 4
- `core_features[1]` — used in Phase 2
- `monetization_model` — recorded in project summary

**Error handling:** If Feasibility check returns FAIL, stop. Revise `problem_domain` or `constraints` and re-run this step before continuing.

---

### Step 2 — Generate Product Name
Execute skill: `skills/project/generate-product-name.md`

Input:
- `concept` → `value_proposition` from Step 1
- `audience` → `market` from playbook input
- `tone_preference` → from playbook input (optional)

Output to capture:
- `display_name` — used in Steps 3, 4, 5, and commit message
- `slug` — used in Step 13 (replace-project-name)
- `env_prefix` — used in Step 13
- `go_package` — used in Step 13

**Condition:** If `product_name` was provided as a playbook input, skip this step and derive `display_name = product_name`, `slug` = kebab-case of product_name, etc.

**Error handling:** If no candidate scores above 6/12, regenerate with a different `style` parameter before proceeding.

---

### Step 3 — Generate Brand
Execute skill: `skills/project/generate-brand.md`

Input:
- `product_name` → `display_name` from Step 2
- `concept` → `value_proposition` from Step 1
- `audience` → `market` from playbook input
- `value_proposition` → from Step 1
- `tone_preference` → from playbook input (optional)

Output to capture:
- `tagline` — used in project summary
- `color_palette` — used in frontend generation (Step 9)
- `tone_adjectives` — passed to generate-frontend-ui for copy consistency
- `positioning_statement` — recorded in project summary

**Error handling:** If validation checklist has any unchecked item, regenerate only the failing section before proceeding.

---

## Phase 2: Specification

**Goal:** Produce a complete, implementation-ready specification for the core feature.

### Step 4 — Generate Core Feature Specification
Execute skill: `skills/saas/generate-core-feature.md`

Input:
- `product_name` → `display_name` from Step 2
- `value_proposition` → from Step 1
- `feature_name` → `core_features[1]` from Step 1
- `target_user` → `market` from playbook input
- `data_entities` → `[]` (empty — this is a new project)

Output to capture:
- `feature_spec` — full specification document
- `api_contract` — used in Step 8 (generate-backend-api)
- `data_model` — used in Step 7 (generate-migration)
- `ui_components` — used in Step 9 (generate-frontend-ui)
- `acceptance_criteria` — used in Step 12 (validate-feature)
- `prerequisites` — verify all are satisfied before Phase 3

**Error handling:** If validation checklist has any unchecked item, fix the spec before proceeding. Do not start implementation from an incomplete spec.

**Checkpoint:** Confirm all items in the spec's prerequisites list are satisfied. If any prerequisite is missing (e.g., auth middleware), document it and add it to Phase 3 before the core feature steps.

---

## Phase 3: Implementation

**Goal:** Generate all implementation artifacts for the core feature.

**Read before implementing:**
- `shared-context/go-clean-architecture.md`
- `shared-context/api-design-guidelines.md`
- `shared-context/database-design-guidelines.md`
- `shared-context/react-frontend-guidelines.md`

**Implementation order is fixed. Do not reorder steps.**

---

### Step 5 — Generate Project Scaffold
Execute skill: `skills/project/replace-project-name.md`

Input:
- `placeholder` → `"my-app"` (default scaffold placeholder)
- `project_name` → `slug` from Step 2
- `display_name` → `display_name` from Step 2
- `env_prefix` → `env_prefix` from Step 2
- `go_package` → `go_package` from Step 2
- `dry_run` → `false`

**Error handling:** If replacements fail, run with `dry_run: true` first to identify the scope. Fix the issue, then re-run with `dry_run: false`. This step is idempotent — it is safe to re-run.

---

### Step 6 — Generate Auth Middleware (prerequisite)
Execute skill: `skills/development/generate-backend-api.md`

Input:
- `resource` → `"auth"`
- `operations` → `["login", "register"]`

**Condition:** Only execute if the core feature spec lists auth as a prerequisite. If auth already exists in the scaffold, skip this step.

**Error handling:** If generation fails, stop. Auth is a hard prerequisite for all protected endpoints. Do not continue to Step 7.

---

### Step 7 — Generate Database Migration
Execute skill: `skills/development/generate-migration.md`

Input:
- `entities` → `data_model` from Step 4
- `is_tenant_scoped` → `true` for all tenant-owned entities

**Error handling:** If migration generation fails, stop. All subsequent steps depend on the correct schema. Fix the entity definition in the spec (Step 4 output) and re-run this step.

---

### Step 8 — Generate Backend API
Execute skill: `skills/development/generate-backend-api.md`

Input:
- `resource` → core feature resource name from spec
- `operations` → operations defined in `api_contract` from Step 4

**Error handling:** If generation fails or produces an incomplete layer (missing handler, use-case, or repository interface), stop and report the missing component. Re-run this step with corrected input before continuing.

---

### Step 9 — Generate Frontend UI
Execute skill: `skills/development/generate-frontend-ui.md`

Input:
- `components` → `ui_components` from Step 4
- `brand` → color palette and tone from Step 3

**Error handling:** If generation fails, stop. Report which component failed and why. Re-run this step only (Step 8 output is safe).

---

### Step 10 — Generate Tests
Execute skill: `skills/development/generate-tests.md`

Input:
- `targets` → use-case and handler names from Step 8
- `frontend_components` → component names from Step 9

**Error handling:** If test generation fails, stop and report. Do not skip this step — untested code must not be committed.

---

## Phase 4: Validation

**Goal:** Verify all generated artifacts are correct before committing.

**All validation skills must return PASS. Any FAIL stops the playbook.**

---

### Step 11 — Validate Database
Execute skill: `skills/validation/validate-database.md`

Validates: migration SQL is syntactically correct, `organization_id` column present on all tenant-scoped tables, no missing indexes on FK columns.

**On FAIL:** Fix the migration file generated in Step 7. Re-run Step 11 only.

---

### Step 12 — Validate Backend
Execute skill: `skills/validation/validate-backend.md`

Validates: layer separation, error handling, context propagation, domain error usage, API response envelopes.

**On FAIL:** Fix the failing file in the layer identified by the validator. Re-run Step 12 only.

---

### Step 13 — Validate Frontend
Execute skill: `skills/validation/validate-frontend.md`

Validates: component structure, API integration points, loading and error states.

**On FAIL:** Fix the failing component from Step 9. Re-run Step 13 only.

---

### Step 14 — Validate Feature
Execute skill: `skills/validation/validate-feature.md`

Validates: all acceptance criteria from Step 4 are implemented and testable.

Input:
- `acceptance_criteria` → from Step 4 spec

**On FAIL:** Identify which acceptance criterion is unmet. Return to the relevant implementation step (7, 8, or 9), fix, and re-run the full validation phase (Steps 11–14).

---

## Phase 5: Repository Setup

**Goal:** Create the GitHub repository and push the validated implementation.

**Only execute this phase after all Phase 4 validations return PASS.**

---

### Step 15 — Create GitHub Repository
Execute skill: `skills/github/create-repo.md`

Input:
- `repo_name` → `slug` from Step 2
- `description` → `tagline` from Step 3
- `visibility` → `"private"`

**Error handling:** If the repository already exists, stop and ask the operator whether to use the existing repo or abort. Do not force-push to an existing repository.

---

### Step 16 — Commit Changes
Execute skill: `skills/github/commit-changes.md`

Input:
- `message` → `"Initial commit: <display_name> SaaS scaffold with <feature_name>"`

**Error handling:** If commit fails due to pre-commit hook, fix the reported issue and re-run the commit. Never use `--no-verify`.

---

## Output

At playbook completion, produce a summary:

```
SaaS Project Summary
────────────────────
Product:          <display_name>
Tagline:          <tagline>
Value Prop:       <value_proposition>
Core Feature:     <feature_name>
Repository:       <github_repo_url>
Commit:           <commit_hash>

Validation Results:
  Database:  PASS
  Backend:   PASS
  Frontend:  PASS
  Feature:   PASS

Generated Files:
  <list of created files>
```
