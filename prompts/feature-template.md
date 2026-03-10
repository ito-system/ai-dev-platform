# Feature Prompt Template

Use this template when prompting Claude to implement a feature.

---

## Role

Read `prompts/system/senior-engineer.md`.

## Context

Project: <project_name>
Repository: <repo_path>
Language/Framework: <language_and_framework>

## Task

Implement the following feature:

**Feature name:** <feature_name>

**User story:**
As a <role>, I want <action> so that <benefit>.

**Acceptance criteria:**
- [ ] <criterion_1>
- [ ] <criterion_2>
- [ ] <criterion_3>

## Constraints

- Read `shared-context/go-clean-architecture.md` before writing backend code.
- Read `shared-context/react-frontend-guidelines.md` before writing frontend code.
- Do not modify files outside the feature scope.
- Run validation skills after implementation.

## Output expected

- Implementation files
- Test files
- Validation report
