# Skill: generate-core-feature

## Responsibility
Generate a complete, implementation-ready specification for the single most critical feature of a SaaS product — the feature that most directly delivers the value proposition.

---

## Input

- `product_name` (required): Name of the SaaS product (e.g., `"OnboardIQ"`)
- `value_proposition` (required): One-sentence value proposition from `generate-tob-saas-idea` output
- `feature_name` (required): Name of the core feature to specify (e.g., `"Automated Task Assignment"`)
- `target_user` (required): Primary user persona (e.g., `"HR manager at a 50-person company"`)
- `data_entities` (optional): Known entities already defined in the project (e.g., `["User", "Organization"]`)

---

## Steps

1. **Read shared architecture documents.**
   - Read `shared-context/go-clean-architecture.md`.
   - Read `shared-context/api-design-guidelines.md`.

2. **Write the user story.**
   - Format: `"As a <target_user>, I want <action> so that <benefit>."`
   - The benefit must directly map to the value proposition.

3. **Define acceptance criteria.**
   - Write a checklist of 4–7 testable conditions.
   - Each criterion must be verifiable without ambiguity (avoid "works correctly" or "looks good").
   - Cover: happy path, validation errors, authorization check, and at least one edge case.

4. **Define the API contract.**
   - List all endpoints required to implement this feature.
   - For each endpoint, specify:
     - Method and path (following `api-design-guidelines.md` URL conventions)
     - Request body fields (name, type, required/optional, validation rule)
     - Success response shape (following canonical `{ "data": ... }` envelope)
     - Relevant error responses (code, HTTP status)
   - Do not generate Go code yet — this is a contract definition only.

5. **Define the data model.**
   - List all domain entities required (new or modified).
   - For each entity, list fields: name, type, constraints (required, unique, FK reference).
   - Identify which entities are **tenant-scoped** (must include `organization_id`).
   - Reference existing `data_entities` to avoid duplication.

6. **Identify downstream skill invocations.**
   - List which development skills will be invoked to implement this spec:
     - `skills/development/generate-migration.md` — for each new or modified entity
     - `skills/development/generate-backend-api.md` — for each endpoint group
     - `skills/development/generate-frontend-ui.md` — for the UI layer
     - `skills/development/generate-tests.md` — for use-case and handler tests

7. **Identify dependencies and prerequisites.**
   - List any features, migrations, or infrastructure that must exist before this feature can be built.
   - Example: "Authentication middleware must be in place before protected endpoints can be implemented."

---

## Output

A complete feature specification document:

```
Core Feature Specification
──────────────────────────
Product:          <product_name>
Feature:          <feature_name>
Target User:      <target_user>

User Story:
  As a <target_user>, I want <action> so that <benefit>.

Acceptance Criteria:
  [ ] <criterion 1>
  [ ] <criterion 2>
  [ ] <criterion 3>
  [ ] <criterion 4>
  [ ] <criterion 5>

API Contract:
  <METHOD> <path>
    Request:  { field: type (required|optional) }
    Response: { "data": { ... } }
    Errors:   { "error": { "code": "...", "message": "..." } }

Data Model:
  Entity: <EntityName> (tenant-scoped: yes/no)
    - id:              string, required, primary key
    - organization_id: string, required, FK → organizations.id  [if tenant-scoped]
    - <field>:         <type>, <constraints>
    - created_at:      timestamp, required
    - updated_at:      timestamp, required

Downstream Skills:
  - generate-migration    → <entity names>
  - generate-backend-api  → <resource names>
  - generate-frontend-ui  → <component names>
  - generate-tests        → <use-case names>

Prerequisites:
  - <prerequisite 1>
  - <prerequisite 2>
```

---

## Validation

- [ ] User story follows the exact format: "As a [user], I want [action] so that [benefit]."
- [ ] Benefit in the user story directly maps to the value proposition.
- [ ] Acceptance criteria contains 4–7 items, all verifiable without ambiguity.
- [ ] At least one acceptance criterion covers an authorization/permission check.
- [ ] At least one acceptance criterion covers a validation error scenario.
- [ ] All API endpoints follow the URL structure from `api-design-guidelines.md`.
- [ ] All response shapes use the canonical `{ "data": ... }` envelope.
- [ ] All tenant-scoped entities include `organization_id`.
- [ ] Downstream skill list is complete — no layer (migration, backend, frontend, tests) is missing.
- [ ] Prerequisites section is populated (or explicitly states "none").

---

## Notes

- This skill specifies **one feature only**. Additional features use `skills/tob-saas/generate-feature.md`.
- The core feature must be the **minimum viable deliverable** that proves the value proposition — do not expand scope.
- Idempotent: if a specification file for this feature already exists, read it first. Only regenerate fields that have changed inputs. Do not overwrite a completed spec without explicit instruction.
- All API endpoint designs must comply with `shared-context/api-design-guidelines.md` before this skill is considered complete.
