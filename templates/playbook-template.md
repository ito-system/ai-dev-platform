# Playbook: <playbook-name>

<!--
  Instructions:
  - Replace <playbook-name> with a verb-noun kebab-case name (e.g., create-saas)
  - Playbooks orchestrate skills. They do not implement logic themselves.
  - Structure: Phase 1 (spec) → Phase 2 (implement) → Phase 3 (validate) → Phase 4 (commit)
  - Place this file in playbooks/<category>/<playbook-name>.md
-->

## Purpose
<!-- One sentence: what workflow does this playbook execute? -->

## Role
Read `prompts/system/<role>.md` before starting.

## Inputs
<!-- List all inputs required to start this playbook -->
- `param_name`: Description

---

## Phase 1: Specification

1. Execute skill: `skills/<category>/<skill>.md`
   - Input: 

---

## Phase 2: Implementation

Read before implementing:
- `shared-context/<doc>.md`

2. Execute skill: `skills/development/generate-backend-api.md`
3. Execute skill: `skills/development/generate-frontend-ui.md`
4. Execute skill: `skills/development/generate-migration.md`
5. Execute skill: `skills/development/generate-tests.md`

---

## Phase 3: Validation

6. Execute skill: `skills/validation/validate-backend.md`
7. Execute skill: `skills/validation/validate-frontend.md`
8. Execute skill: `skills/validation/validate-database.md`
9. Execute skill: `skills/validation/validate-feature.md`

**Stop if any validation returns FAIL.**

---

## Phase 4: Commit

10. Execute skill: `skills/github/commit-changes.md`
    - Message: `"<commit message>"`

---

## Output
<!-- What does this playbook produce at the end? -->
- 
