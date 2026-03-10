# Skill: generate-feature

## Responsibility
Generate a complete, implementation-ready specification for an additional feature of an existing SaaS product, ensuring it does not break existing functionality.

---

## Input

- `product_name` (required): Name of the SaaS product (e.g., `"OnboardIQ"`)
- `feature_name` (required): Name of the feature to specify (e.g., `"Email Notification Digest"`)
- `user_story` (required): Seed story in the format `"As a <role>, I want <action> so that <benefit>."`
- `existing_entities` (required): List of domain entities already defined in the project (e.g., `["User", "Organization", "Task"]`)
- `existing_endpoints` (required): List of existing API endpoints (method + path) to avoid conflicts
- `priority` (optional, default: `"standard"`): Feature priority — `"critical"`, `"standard"`, `"low"` — affects scope guidance

---

## Steps

1. **Read shared architecture documents.**
   - Read `shared-context/go-clean-architecture.md`.
   - Read `shared-context/api-design-guidelines.md`.

2. **Validate the user story format.**
   - Confirm it follows: `"As a <role>, I want <action> so that <benefit>."`
   - If the format is wrong, rewrite it to conform before proceeding.

3. **Expand acceptance criteria.**
   - Write 4–8 testable acceptance criteria covering:
     - Happy path (feature works as expected)
     - Input validation (required fields, format constraints)
     - Authorization (only permitted users can access)
     - Isolation (feature does not expose data across tenants)
     - At least one edge case (empty state, concurrent operations, boundary values)
   - Each criterion must be independently verifiable.

4. **Define new or modified API endpoints.**
   - Cross-reference `existing_endpoints` to ensure no path conflicts.
   - For each new endpoint, specify:
     - Method and path (following `api-design-guidelines.md` conventions)
     - Request body or query parameters (name, type, required/optional, validation)
     - Success response shape using canonical `{ "data": ... }` envelope
     - Error responses (code, HTTP status, scenario)
   - For modified endpoints, list only the delta — what changes vs. what stays the same.

5. **Define data model changes.**
   - Cross-reference `existing_entities` to identify reuse vs. new entities.
   - For new entities: list all fields (name, type, constraints, tenant-scope).
   - For modified entities: list only added or changed fields — never remove fields in this step.
   - Identify all required database migrations (new table, new column, new index).

6. **List UI components.**
   - List each UI component required to implement the feature.
   - For each component: name, type (page / section / modal / form / table / button), and a one-sentence description of its purpose.
   - Reference existing components where applicable — do not re-specify what already exists.

7. **Identify edge cases and error states.**
   - List at least 3 specific edge cases that the implementation must handle.
   - Format: `Scenario — Expected behavior`
   - Examples:
     - "User submits form with duplicate email — return 409 CONFLICT with field-level error."
     - "Feature list is empty on first load — display empty state with call-to-action."
     - "Concurrent update by two users — last-write-wins, no silent data loss."

8. **Assess impact on existing features.**
   - Explicitly state which existing endpoints, entities, or UI components are touched.
   - If a breaking change is required, flag it as `[BREAKING]` and describe the migration path.
   - If no existing features are affected, state: "No impact on existing features."

9. **Identify downstream skill invocations.**
   - `skills/development/generate-migration.md` — for each new or modified entity
   - `skills/development/generate-backend-api.md` — for each new endpoint group
   - `skills/development/generate-frontend-ui.md` — for each new UI component group
   - `skills/development/generate-tests.md` — for the new use-case logic

---

## Output

A complete feature specification document:

```
Feature Specification
─────────────────────
Product:      <product_name>
Feature:      <feature_name>
Priority:     <priority>

User Story:
  As a <role>, I want <action> so that <benefit>.

Acceptance Criteria:
  [ ] <criterion — happy path>
  [ ] <criterion — validation>
  [ ] <criterion — authorization>
  [ ] <criterion — tenant isolation>
  [ ] <criterion — edge case>

API Changes:
  NEW   <METHOD> <path>
    Request:  { field: type (required|optional, validation) }
    Response: { "data": { ... } }
    Errors:   [{ code, status, scenario }]

  MODIFIED  <METHOD> <path>
    Delta: <what changes>

Data Model Changes:
  NEW Entity: <EntityName> (tenant-scoped: yes/no)
    - <field>: <type>, <constraints>

  MODIFIED Entity: <EntityName>
    + <new_field>: <type>, <constraints>

  Required Migrations:
    - <migration description>

UI Components:
  - <ComponentName> (<type>): <purpose>

Edge Cases:
  - <Scenario> — <Expected behavior>
  - <Scenario> — <Expected behavior>
  - <Scenario> — <Expected behavior>

Impact on Existing Features:
  <description or "No impact on existing features.">
  [BREAKING] <description + migration path>  (if applicable)

Downstream Skills:
  - generate-migration    → <entity names>
  - generate-backend-api  → <resource names>
  - generate-frontend-ui  → <component names>
  - generate-tests        → <use-case names>
```

---

## Validation

- [ ] User story follows the exact format: "As a [role], I want [action] so that [benefit]."
- [ ] Acceptance criteria: 4–8 items, each independently verifiable.
- [ ] At least one criterion covers tenant isolation.
- [ ] No new API endpoint path conflicts with `existing_endpoints`.
- [ ] All new API paths follow `api-design-guidelines.md` URL conventions.
- [ ] All response shapes use the canonical `{ "data": ... }` envelope.
- [ ] All new tenant-scoped entities include `organization_id`.
- [ ] All new entities cross-referenced against `existing_entities` — no duplicates.
- [ ] At least 3 edge cases listed with expected behavior.
- [ ] Impact on existing features is explicitly stated (even if "no impact").
- [ ] Any breaking change is labeled `[BREAKING]` with a migration path.
- [ ] Downstream skills list is complete for all affected layers.

---

## Notes

- This skill specifies **one feature only**. Repeat invocations for each additional feature.
- `priority: "critical"` requires all edge cases to have explicit handling defined. Lower priorities may document edge cases as "future scope."
- Idempotent: if a specification for `feature_name` already exists, read it first. Only update sections where the inputs differ. Do not overwrite an existing spec without explicit instruction.
- Features must be **independently deployable** — the product must remain functional if this feature is toggled off.
- Do not generate code. Code generation is delegated to `skills/development/` skills.
