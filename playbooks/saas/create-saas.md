# Playbook: create-saas

## Purpose
Generate a complete SaaS application from a product concept.

## Role
Read `prompts/system/architect-role.md` before starting.

## Inputs
- `market`: Target market (e.g., "HR teams at SMBs")
- `problem_domain`: Problem to solve
- `product_name` (optional): Pre-defined name, or generate one

---

## Phase 1: Ideation

1. Execute skill: `skills/saas/generate-saas-idea.md`
   - Input: `market`, `problem_domain`
2. Execute skill: `skills/project/generate-product-name.md`
   - Input: concept from step 1
3. Execute skill: `skills/project/generate-brand.md`
   - Input: product name, concept, audience

---

## Phase 2: Specification

4. Execute skill: `skills/saas/generate-core-feature.md`
   - Input: product name, value proposition, target user

---

## Phase 3: Implementation

Read before implementing:
- `shared-context/go-clean-architecture.md`
- `shared-context/api-design-guidelines.md`
- `shared-context/database-design-guidelines.md`
- `shared-context/react-frontend-guidelines.md`

5. Execute skill: `skills/development/generate-migration.md`
   - Input: entities from core feature spec
6. Execute skill: `skills/development/generate-backend-api.md`
   - Input: resource from core feature spec
7. Execute skill: `skills/development/generate-frontend-ui.md`
   - Input: UI components from core feature spec
8. Execute skill: `skills/development/generate-tests.md`
   - Input: backend use-case and frontend component

---

## Phase 4: Validation

9. Execute skill: `skills/validation/validate-database.md`
10. Execute skill: `skills/validation/validate-backend.md`
11. Execute skill: `skills/validation/validate-frontend.md`
12. Execute skill: `skills/validation/validate-feature.md`

**Do not proceed past Phase 4 if any validation returns FAIL.**

---

## Phase 5: Repository Setup

13. Execute skill: `skills/github/create-repo.md`
14. Execute skill: `skills/project/replace-project-name.md`
15. Execute skill: `skills/github/commit-changes.md`
    - Message: `"Initial commit: <product_name> SaaS scaffold"`

---

## Output
- GitHub repository URL
- Product name and brand summary
- Core feature specification
- Implementation file list
- Validation report
