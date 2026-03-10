# Feature Specification: <feature-name>

<!--
  Instructions:
  - Fill in all sections before passing this spec to an implementation playbook.
  - Acceptance criteria must be testable and unambiguous.
-->

## Metadata

- **Product:** <product_name>
- **Feature name:** <feature_name>
- **Author:** <author>
- **Date:** <YYYY-MM-DD>
- **Status:** Draft / Ready / Implemented

---

## User story

As a **<role>**, I want **<action>** so that **<benefit>**.

---

## Acceptance criteria

- [ ] <criterion_1>
- [ ] <criterion_2>
- [ ] <criterion_3>

---

## API contract

### Endpoints

| Method | Path | Description |
|---|---|---|
| POST | /api/v1/<resource> | Create <resource> |
| GET | /api/v1/<resource>/{id} | Get <resource> by ID |

### Request body (POST)
```json
{
  "field_name": "value"
}
```

### Response (200)
```json
{
  "data": {
    "id": "uuid",
    "field_name": "value"
  }
}
```

---

## Data model

### New tables

| Table | Columns |
|---|---|
| `<table_name>` | `id`, `field_name`, `created_at`, `updated_at` |

### Modified tables

| Table | Change |
|---|---|
| `<table_name>` | Add column `<column_name>` |

---

## UI components

- `<ComponentName>`: Description

---

## Edge cases

- <edge_case_1>
- <edge_case_2>

---

## Out of scope

- <exclusion_1>
