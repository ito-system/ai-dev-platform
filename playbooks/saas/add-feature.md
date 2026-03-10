# Playbook: add-feature (SaaS)

## Purpose
Add a new feature to an existing SaaS product.

## Role
Read `prompts/system/senior-engineer.md` before starting.

## Inputs
- `product_name`: Name of the SaaS product
- `feature_name`: Name of the feature to add
- `user_story`: "As a <role>, I want <action> so that <benefit>"
- `repo_path`: Path to the local repository

---

## Phase 1: Specification

1. Execute skill: `skills/saas/generate-feature.md`
   - Input: product name, feature name, user story

---

## Phase 2: Implementation

Read before implementing:
- `shared-context/go-clean-architecture.md`
- `shared-context/api-design-guidelines.md`
- `shared-context/database-design-guidelines.md`
- `shared-context/react-frontend-guidelines.md`

2. Execute skill: `skills/development/generate-migration.md` (if schema changes required)
3. Execute skill: `skills/development/generate-backend-api.md`
4. Execute skill: `skills/development/generate-frontend-ui.md`
5. Execute skill: `skills/development/generate-tests.md`

---

## Phase 3: Validation

6. Execute skill: `skills/validation/validate-database.md` (if migrations generated)
7. Execute skill: `skills/validation/validate-backend.md`
8. Execute skill: `skills/validation/validate-frontend.md`
9. Execute skill: `skills/validation/validate-feature.md`

**Stop if any validation returns FAIL.**

---

## Phase 4: Commit

10. Execute skill: `skills/github/commit-changes.md`
    - Message: `"feat: add <feature_name>"`

---

## Output
- Feature specification document
- Implementation files
- Validation report
- Commit hash
