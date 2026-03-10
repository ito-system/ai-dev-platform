# Skill: generate-feature-spec

## Responsibility
Generate a complete, structured feature specification document — covering database, backend, frontend, tests, edge cases, and seed data — that Playbooks can consume directly to produce production-ready code without further clarification.

---

## Input

- `product_name` (required): Display name of the SaaS product (e.g., `"OnboardIQ"`)
- `feature_name` (required): Name of the feature to specify (e.g., `"Email Notification Digest"`)
- `user_story` (required): `"As a <role>, I want <action> so that <benefit>."`
- `existing_entities` (required): List of current domain entities (e.g., `["User", "Organization", "Task"]`)
- `existing_endpoints` (required): List of current API endpoints as `"METHOD /path"` strings (e.g., `["GET /api/v1/users", "POST /api/v1/tasks"]`)
- `last_migration_number` (required): Integer — the last applied migration number (e.g., `5`)
- `priority` (optional, default: `"standard"`): `"critical"` | `"standard"` | `"low"`

---

## Steps

### Step 1 — Read Shared Context

Read the following documents in order. Do not skip any:

1. `shared-context/go-clean-architecture.md` — layer rules, naming conventions, context propagation, multi-tenancy
2. `shared-context/api-design-guidelines.md` — endpoint patterns, response envelopes, error codes, pagination
3. `shared-context/database-design-guidelines.md` — schema conventions, migration numbering, FK and index rules
4. `shared-context/react-frontend-guidelines.md` — component structure, API integration patterns, state management

All specification decisions must conform to these documents. If a conflict arises between the feature request and a shared-context rule, the shared-context rule takes precedence. Document the conflict explicitly in the spec.

---

### Step 2 — Validate and Normalize Inputs

1. **Validate user story format.**
   - Must match: `"As a <role>, I want <action> so that <benefit>."`
   - If the format is invalid, rewrite it to conform before continuing.

2. **Resolve existing state.**
   - Parse `existing_entities` into a lookup set.
   - Parse `existing_endpoints` into a lookup set of `METHOD /path` pairs.
   - Record `last_migration_number` as the baseline for numbering new migrations.

3. **Classify the feature scope.**
   - Determine which layers are affected: database / backend / frontend / tests.
   - If only one layer is affected, mark the others as `NOT APPLICABLE` in the output.

---

### Step 3 — Detect Conflicts with Existing Features

1. **Entity name conflicts.**
   - List all entity names the feature will introduce.
   - For each: if the name already exists in `existing_entities`, flag as `[CONFLICT]` and propose a resolution (rename or reuse existing entity).

2. **Endpoint conflicts.**
   - List all new `METHOD /path` pairs the feature will introduce.
   - For each: if the exact pair exists in `existing_endpoints`, flag as `[CONFLICT]`.
   - If the path exists with a different method (e.g., existing `GET /api/v1/tasks`, new `POST /api/v1/tasks`), this is acceptable — document the rationale.

3. **Breaking change detection.**
   - A breaking change occurs when any of the following would result from this feature:
     - Removing or renaming a field from an existing response
     - Changing a field's type in an existing entity or endpoint
     - Dropping a database column or table used by existing features
     - Changing an existing function signature in a shared use-case or handler
   - For each detected breaking change: flag as `[BREAKING]` and provide the required migration path.

**Output of this step:** A conflict and breaking-change report. If any `[CONFLICT]` is unresolved, stop. Do not proceed to Step 4 until all conflicts are resolved.

---

### Step 4 — Generate Database Specification

For each new or modified database entity:

**New entity:**
- Entity name (PascalCase)
- Tenant-scoped: yes / no (if yes, include `organization_id UUID NOT NULL REFERENCES organizations(id)`)
- Fields: name, PostgreSQL type, constraints (NOT NULL, UNIQUE, DEFAULT, FK reference)
- Indexes: define index for every FK column and every column used in WHERE clauses
- `created_at TIMESTAMPTZ NOT NULL DEFAULT now()`
- `updated_at TIMESTAMPTZ NOT NULL DEFAULT now()`
- Migration number: `last_migration_number + 1` (increment for each new migration file)
- Migration filename: `<migration_number>_create_<snake_case_entity_name>.sql`

**Modified entity:**
- Entity name
- Changes: list of `ALTER TABLE` statements only (no DROP COLUMN unless explicitly required and tagged `[BREAKING]`)
- Migration number: next available after new-entity migrations
- Migration filename: `<migration_number>_alter_<snake_case_entity_name>_<description>.sql`

**Seed data requirements:**
- For each new entity, define the minimum seed records needed to populate the dashboard with non-sensitive dummy data.
- Format: `Table: <table_name> — <N> records — fields: <field>=<value>, ...`
- Seed data must use only non-sensitive, clearly fictional values (e.g., `user@example.com`, `Test Organization`).
- Seed data must be idempotent (check for existence before inserting).

---

### Step 5 — Generate API Specification

For each new endpoint:

```
<METHOD> <path>
  Description: <one sentence>
  Auth:        required | public
  Request:
    Body:      { field: type (required|optional) — validation rule }
    Query:     ?param=type (optional|required) — description
  Response:
    200/201:   { "data": { field: type, ... } }
    Errors:
      400:     { "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [...] } }
      401:     { "error": { "code": "UNAUTHORIZED", "message": "..." } }
      404:     { "error": { "code": "NOT_FOUND", "message": "..." } }
```

For each modified endpoint:
- Specify only the delta (added fields, changed behavior).
- Prefix with `MODIFIED:` and list what changes vs. what is unchanged.
- If the modification is breaking, label `[BREAKING]` and specify the migration path.

Validate every endpoint against `api-design-guidelines.md`:
- [ ] URL uses plural noun, kebab-case
- [ ] HTTP method matches the operation semantics
- [ ] Response uses `{ "data": ... }` or `{ "error": ... }` envelope
- [ ] Correct HTTP status code for each scenario

---

### Step 6 — Generate Backend Specification

For each new use-case method:

```
UseCase: <ResourceUseCase>
Method:  <MethodName>(ctx context.Context, <params>) (<return>, error)
  Logic:
    1. <step>
    2. <step>
  Calls:    <RepositoryMethod>(ctx, ...)
  Returns:  *domain.<Entity> | domain.Err<Type>
  Tests:
    - Happy path: <description>
    - Error: <domain error> → <expected behavior>
```

For each new repository method:

```
Repository interface method: <MethodName>(ctx context.Context, <params>) (<return>, error)
  Query intent:  <SELECT/INSERT/UPDATE/DELETE — what it does>
  Tenant filter: organization_id extracted from ctx (if tenant-scoped)
  SQL pattern:   <brief description of WHERE clause>
```

For each new HTTP handler:

```
Handler: <ResourceHandler>.<MethodName>(w, r)
  Parses:   <fields from request body or path params>
  Calls:    <UseCaseMethod>
  Writes:   <HTTP status code> { "data": { ... } }
  On error: maps domain errors to HTTP codes per api-design-guidelines.md
```

---

### Step 7 — Generate Frontend Specification

For each new component or page:

```
Component: <ComponentName>
  Type:      page | section | modal | form | table | button
  Route:     /path (if page)
  Purpose:   <one sentence>
  API calls:
    - <METHOD> <path> — on <event/mount>
  State:
    - loading: bool — shown while API call is in-flight
    - error:   string | null — shown if API call fails
    - data:    <type> | null — populated on success
  Props:      <prop: type (required|optional)>
  Empty state: <what to render when data is empty>
  Error state: <what to render when API call fails>
```

For each modified component:
- List only the changes (added props, new API calls, modified state).
- Do not re-specify unchanged behavior.

---

### Step 8 — Generate Test Specification

For each new use-case method:

```
Test file: internal/usecase/<resource>_usecase_test.go
  Test: <MethodName>_HappyPath
    Setup:   mock repo returns <value>
    Input:   <params>
    Expects: <return value>

  Test: <MethodName>_NotFound
    Setup:   mock repo returns domain.ErrNotFound
    Input:   <params>
    Expects: domain.ErrNotFound returned

  Test: <MethodName>_<EdgeCase>
    Setup:   <mock setup>
    Input:   <edge case params>
    Expects: <expected behavior>
```

For each new HTTP handler:

```
Test file: internal/handler/<resource>_handler_test.go
  Test: <MethodName>_Returns200
    Request:  <method> <path> with valid body
    Mock:     use-case returns <value>
    Expects:  status 200, body { "data": { ... } }

  Test: <MethodName>_Returns400_ValidationError
    Request:  <method> <path> with invalid body
    Expects:  status 400, body { "error": { "code": "VALIDATION_ERROR", ... } }

  Test: <MethodName>_Returns404
    Mock:     use-case returns domain.ErrNotFound
    Expects:  status 404, body { "error": { "code": "NOT_FOUND", ... } }
```

For each new frontend component:

```
Test file: <ComponentName>.test.tsx
  Test: renders loading state
  Test: renders error state when API fails
  Test: renders data on success
  Test: <edge case behavior>
```

**Edge case tests** (for all `priority: "critical"` features, all edge cases must have test specifications):
- List each edge case with: scenario, input condition, expected system behavior.

---

### Step 9 — Write Acceptance Criteria

Write 5–10 acceptance criteria:

```
Acceptance Criteria:
  [ ] <criterion — happy path>
  [ ] <criterion — validation>
  [ ] <criterion — authorization>
  [ ] <criterion — tenant isolation: feature only exposes data within the user's organization>
  [ ] <criterion — edge case>
  [ ] <criterion — test coverage: all use-case methods have unit tests>
  [ ] <criterion — API contract: all responses follow canonical envelope format>
```

Each criterion must be:
- Independently verifiable
- Phrased as a testable statement (not "works correctly" or "looks good")

---

### Step 10 — Assess Impact on Existing Features

State explicitly for each category:

- **Existing endpoints affected:** list or "none"
- **Existing entities modified:** list or "none"
- **Existing UI components modified:** list or "none"
- **Breaking changes:** list with `[BREAKING]` tag and migration path, or "none"
- **Overall impact level:** `isolated` (no existing code modified) | `additive` (existing code extended without breaking) | `breaking` (requires migration path)

---

### Step 11 — Save Specification Document

Save the complete specification to:

```
docs/feature-specs/<kebab-case-feature-name>.md
```

**Idempotency guard:**
- Check whether `docs/feature-specs/<kebab-case-feature-name>.md` already exists.
- If it exists: read it first. Compare the existing spec's `user_story` and `feature_name` with the current inputs.
  - If unchanged: log `SKIP — spec already exists and inputs are unchanged`. Do not overwrite.
  - If changed: overwrite with the new spec and note the change at the top of the document: `Updated: <timestamp> — <what changed>`.
- If it does not exist: create the file.

---

## Output

A saved specification file at `docs/feature-specs/<kebab-case-feature-name>.md` with the following structure:

```markdown
# Feature Spec: <feature_name>

Product:   <product_name>
Priority:  <priority>
Created:   <ISO 8601 date>

## User Story
As a <role>, I want <action> so that <benefit>.

## Conflict Report
<CONFLICT or BREAKING findings, or "No conflicts detected.">

## Database Changes
### New Entities
  <entity spec>
### Modified Entities
  <entity spec or "none">
### Migrations
  <migration N: filename — description>
### Seed Data
  <table: N records — fields>

## API Changes
### New Endpoints
  <endpoint spec>
### Modified Endpoints
  <delta spec or "none">

## Backend Specification
### Use-Cases
  <use-case spec>
### Repository Methods
  <repository spec>
### Handlers
  <handler spec>

## Frontend Specification
### Components / Pages
  <component spec>

## Test Specification
### Use-Case Tests
  <test spec>
### Handler Tests
  <test spec>
### Frontend Tests
  <test spec>
### Edge Cases
  <edge case: scenario — expected behavior>

## Acceptance Criteria
  [ ] <criterion>

## Impact Assessment
  Existing endpoints affected: <list or none>
  Existing entities modified:  <list or none>
  Existing UI modified:        <list or none>
  Breaking changes:            <list or none>
  Overall impact level:        isolated | additive | breaking
```

**Structured outputs for Playbook consumption:**

| Output key | Value |
|---|---|
| `feature_spec` | Path to saved spec file |
| `api_changes` | List of new and modified endpoint specs |
| `data_model_changes` | New/modified entities + migration list |
| `ui_components` | Component/page names to implement |
| `edge_cases` | Edge case definitions |
| `acceptance_criteria` | Acceptance criteria checklist |
| `impact_assessment` | Impact level + affected artifacts |
| `breaking_changes` | `[BREAKING]` items with migration paths |
| `seed_requirements` | Seed data spec per table |

---

## Validation

- [ ] User story conforms to the required format.
- [ ] All shared-context documents were read before generating the spec.
- [ ] No unresolved `[CONFLICT]` items remain in the conflict report.
- [ ] All new API endpoints are validated against `api-design-guidelines.md` checklist.
- [ ] All new tenant-scoped entities include `organization_id` with FK.
- [ ] Migration numbers are sequential starting from `last_migration_number + 1`.
- [ ] Each new use-case method has at minimum: happy path, not-found, and one edge-case test spec.
- [ ] Each new handler has at minimum: 200, 400, and 404 test specs.
- [ ] Each new frontend component has: loading state, error state, and success state test specs.
- [ ] Acceptance criteria: 5–10 items, all independently verifiable.
- [ ] Impact assessment explicitly states the overall impact level.
- [ ] Spec file saved to `docs/feature-specs/<kebab-case-feature-name>.md`.
- [ ] Idempotency guard checked before writing the file.

---

## Notes

- This skill generates **specification only** — no code is written.
- The saved spec file is the single source of truth for this feature. Playbooks must read it before starting implementation.
- Seed data records must use clearly fictional values. Never use real email addresses, real organization names, or any PII.
- If `priority` is `"critical"`, all edge cases must have test specifications. For `"standard"` and `"low"`, document edge cases in the spec but mark test coverage as recommended rather than required.
- This skill supersedes `skills/saas/generate-feature.md` when a saved spec file is required. Use `generate-feature.md` for lightweight, in-session-only specs. Use this skill when the spec must persist and be consumed by downstream Playbooks.
