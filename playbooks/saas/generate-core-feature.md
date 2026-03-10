# Playbook: generate-core-feature (SaaS)

## Purpose
Generate and implement the core feature of an existing SaaS product.

## Role
Read `prompts/system/senior-engineer.md` before starting.

## Inputs
- `product_name`: Name of the SaaS product
- `value_proposition`: One-sentence value proposition
- `target_user`: Primary user persona
- `repo_path`: Path to the local repository

---

## Phase 1: Specification

1. Execute skill: `skills/saas/generate-core-feature.md`
   - Input: product name, value proposition, target user

---

## Phase 2: Implementation

Read before implementing:
- `shared-context/go-clean-architecture.md`
- `shared-context/api-design-guidelines.md`
- `shared-context/database-design-guidelines.md`
- `shared-context/react-frontend-guidelines.md`

2. Execute skill: `skills/development/generate-migration.md`
3. Execute skill: `skills/development/generate-backend-api.md`
4. Execute skill: `skills/development/generate-frontend-ui.md`
5. Execute skill: `skills/development/generate-tests.md`

---

## Phase 3: Validation

6. Execute skill: `skills/validation/validate-database.md`
7. Execute skill: `skills/validation/validate-backend.md`
8. Execute skill: `skills/validation/validate-frontend.md`
9. Execute skill: `skills/validation/validate-feature.md`

**Stop if any validation returns FAIL.**

---

## Phase 4: Commit

10. Execute skill: `skills/github/commit-changes.md`
    - Message: `"Add core feature: <feature_name>"`

---

## Output
- Core feature specification document
- Implementation files
- Validation report
- Commit hash
