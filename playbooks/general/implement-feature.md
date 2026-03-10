# Playbook: implement-feature

## Purpose
Implement a feature in any project type (not SaaS-specific).

## Role
Read `prompts/system/senior-engineer.md` before starting.

## Inputs
- `project_name`: Name of the project
- `feature_name`: Name of the feature
- `user_story`: "As a <role>, I want <action> so that <benefit>"
- `repo_path`: Path to the local repository

---

## Phase 1: Specification

1. Execute skill: `skills/tob-saas/generate-feature.md`

---

## Phase 2: Implementation

Read relevant shared-context documents before implementing.

2. Execute skill: `skills/development/generate-backend-api.md` (if API required)
3. Execute skill: `skills/development/generate-frontend-ui.md` (if UI required)
4. Execute skill: `skills/development/generate-migration.md` (if DB changes required)
5. Execute skill: `skills/development/generate-tests.md`

---

## Phase 3: Validation

6. Execute skill: `skills/validation/validate-feature.md`
7. Execute skill: `skills/validation/validate-backend.md` (if backend changed)
8. Execute skill: `skills/validation/validate-frontend.md` (if frontend changed)

**Stop if any validation returns FAIL.**

---

## Phase 4: Commit

9. Execute skill: `skills/github/commit-changes.md`
   - Message: `"feat: <feature_name>"`

---

## Output
- Feature specification
- Implementation files
- Validation report
- Commit hash
