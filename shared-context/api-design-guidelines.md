# API Design Guidelines

This document defines REST API design standards for all projects on this platform.
All backend API generation skills must follow this guide without deviation.

---

## Core Principles

- APIs are **contracts**. Once published, breaking changes require versioning.
- URLs identify **resources**, not actions.
- HTTP methods express the action.
- Responses are always **enveloped** in a consistent structure.
- Errors are always **structured** and machine-readable.

---

## URL Structure

### Base pattern

```
/api/v1/{resources}
/api/v1/{resources}/{id}
/api/v1/{resources}/{id}/{sub-resources}
/api/v1/{resources}/{id}/{sub-resources}/{sub-id}
```

### HTTP method mapping

| Method | URL pattern | Meaning |
|---|---|---|
| GET | `/api/v1/users` | List users |
| GET | `/api/v1/users/{id}` | Get one user |
| POST | `/api/v1/users` | Create a user |
| PUT | `/api/v1/users/{id}` | Replace a user (full update) |
| PATCH | `/api/v1/users/{id}` | Update specific fields |
| DELETE | `/api/v1/users/{id}` | Delete a user |

### Naming rules

- Resource names are **plural nouns**: `/users`, `/organizations`, `/invoices`, `/payment-methods`.
- Use **kebab-case** for multi-word resources: `/api/v1/payment-methods`, `/api/v1/audit-logs`.
- **Never use verbs** in URLs. Actions are expressed via HTTP methods.
- Nested resources are allowed up to **one level of nesting**:
  - Allowed: `/api/v1/organizations/{id}/members`
  - Avoid: `/api/v1/organizations/{id}/teams/{teamId}/members/{memberId}/roles`

### Examples

```
GET    /api/v1/users
GET    /api/v1/users/{id}
POST   /api/v1/users
PATCH  /api/v1/users/{id}
DELETE /api/v1/users/{id}

GET    /api/v1/organizations
GET    /api/v1/organizations/{id}
POST   /api/v1/organizations
GET    /api/v1/organizations/{id}/members
POST   /api/v1/organizations/{id}/members
DELETE /api/v1/organizations/{id}/members/{userId}

GET    /api/v1/subscriptions
GET    /api/v1/invoices/{id}
GET    /api/v1/payment-methods
```

---

## HTTP Status Codes

Use the following codes consistently. Do not use other 4xx or 5xx codes unless there is a documented reason.

| Situation | Code | Notes |
|---|---|---|
| Success — read or update | 200 | GET, PUT, PATCH |
| Success — created | 201 | POST that creates a resource |
| Success — no body | 204 | DELETE |
| Validation error | 400 | Missing or malformed input fields |
| Unauthenticated | 401 | No valid token provided |
| Forbidden | 403 | Token is valid but lacks permission |
| Not found | 404 | Resource does not exist |
| Conflict | 409 | Duplicate resource or state conflict |
| Internal server error | 500 | Unexpected failure — log and investigate |

---

## Response Envelope

All responses use one of three envelopes. **Never return bare objects or arrays.**

### Single resource — success

```json
{
  "data": {
    "id": "usr_01J9XK...",
    "email": "alice@example.com",
    "role": "admin",
    "created_at": "2025-03-11T09:00:00Z"
  }
}
```

### List resource — success

```json
{
  "data": [
    {
      "id": "usr_01J9XK...",
      "email": "alice@example.com",
      "role": "admin",
      "created_at": "2025-03-11T09:00:00Z"
    }
  ],
  "meta": {
    "total": 84,
    "page": 1,
    "per_page": 20,
    "total_pages": 5
  }
}
```

### Error — all failure responses

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed.",
    "details": [
      {
        "field": "email",
        "message": "must be a valid email address"
      },
      {
        "field": "role",
        "message": "must be one of: admin, member, viewer"
      }
    ]
  }
}
```

For non-validation errors, `details` is omitted:

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found."
  }
}
```

---

## Error Codes

Error codes are SCREAMING_SNAKE_CASE strings. Use the following standard codes:

| Code | HTTP Status | When to use |
|---|---|---|
| `VALIDATION_ERROR` | 400 | One or more request fields are invalid |
| `INVALID_REQUEST` | 400 | Malformed JSON or missing required body |
| `UNAUTHORIZED` | 401 | No valid authentication token |
| `FORBIDDEN` | 403 | Authenticated but not permitted |
| `NOT_FOUND` | 404 | Resource does not exist |
| `CONFLICT` | 409 | Duplicate entry or state conflict |
| `INTERNAL_ERROR` | 500 | Unexpected server failure |

Do not expose internal error messages or stack traces in the `message` field for 500 errors.

---

## Validation Error Structure

Validation errors (400) **must** include a `details` array. Each item identifies the failing field and a human-readable reason.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed.",
    "details": [
      {
        "field": "email",
        "message": "is required"
      },
      {
        "field": "role",
        "message": "must be one of: admin, member, viewer"
      }
    ]
  }
}
```

- `field` — the JSON field name from the request body (dot-notation for nested fields: `"address.zip"`).
- `message` — a lowercase, human-readable description of the failure. Do not use technical jargon.

---

## Pagination

All list endpoints support cursor-based or page-based pagination via query parameters.

### Query parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | 1 | Page number (1-indexed) |
| `per_page` | integer | 20 | Items per page (max: 100) |

Example:
```
GET /api/v1/users?page=2&per_page=50
```

### Response meta

```json
{
  "data": [ ... ],
  "meta": {
    "total": 84,
    "page": 2,
    "per_page": 50,
    "total_pages": 2
  }
}
```

---

## Query Parameter Naming

- All query parameter names use **snake_case**: `?sort_by=created_at`, `?filter_role=admin`.
- Boolean parameters use `true` / `false` strings: `?include_deleted=true`.
- Date range filters: `?created_after=2025-01-01&created_before=2025-03-01` (ISO 8601).
- Sorting: `?sort_by=created_at&sort_order=desc` (default: `asc`).

---

## JSON Field Naming

- All JSON field names use **snake_case**: `created_at`, `organization_id`, `is_active`.
- Timestamps use ISO 8601 format with UTC timezone: `"2025-03-11T09:00:00Z"`.
- IDs are strings, not integers. Use prefixed IDs (e.g., `"usr_..."`, `"org_..."`) for clarity.
- Boolean fields use the form `is_*` or `has_*`: `is_active`, `has_subscription`.
- Never use abbreviations in field names: `organization_id` not `org_id`, `created_at` not `created`.

---

## Authentication

- All protected endpoints require a `Bearer` token in the `Authorization` header.
- Format: `Authorization: Bearer <token>`
- Never accept credentials in URL query parameters.
- Missing or invalid token → 401 `UNAUTHORIZED`.
- Valid token but insufficient permissions → 403 `FORBIDDEN`.

---

## Request Body

- Content type: `application/json`.
- Handlers must validate the `Content-Type` header on POST/PUT/PATCH requests.
- Unknown fields in the request body are silently ignored (do not return 400).
- Required fields with missing values → 400 `VALIDATION_ERROR` with field-level details.

---

## Versioning

- Version is included in the URL path: `/api/v1/`, `/api/v2/`.
- A new major version is required for any breaking change.
- Breaking changes include:
  - Removing a field from a response
  - Changing a field type
  - Removing or renaming an endpoint
  - Changing required fields
- Non-breaking additions (new optional fields, new endpoints) do not require a version bump.
- Old versions must remain operational until all clients have migrated. Document deprecation in the response via a `Deprecation` header.

---

## Route Registration Conventions

Routes are registered in `routes/routes.go`. Group routes by resource and apply middleware consistently.

```go
// routes/routes.go
func Setup(userH *handler.UserHandler, orgH *handler.OrganizationHandler, authMW, tenantMW func(http.Handler) http.Handler) http.Handler {
    mux := http.NewServeMux()

    // Public routes (no auth)
    mux.HandleFunc("POST /api/v1/auth/login", authH.Login)
    mux.HandleFunc("POST /api/v1/auth/register", authH.Register)

    // Protected routes
    protected := http.NewServeMux()
    protected.HandleFunc("GET  /api/v1/users",          userH.List)
    protected.HandleFunc("GET  /api/v1/users/{id}",     userH.GetByID)
    protected.HandleFunc("POST /api/v1/users",          userH.Create)
    protected.HandleFunc("PATCH /api/v1/users/{id}",    userH.Update)
    protected.HandleFunc("DELETE /api/v1/users/{id}",   userH.Delete)

    protected.HandleFunc("GET  /api/v1/organizations",      orgH.List)
    protected.HandleFunc("POST /api/v1/organizations",      orgH.Create)
    protected.HandleFunc("GET  /api/v1/organizations/{id}", orgH.GetByID)

    mux.Handle("/api/v1/", tenantMW(authMW(protected)))
    return mux
}
```

---

## CORS and Headers

- Set `Content-Type: application/json` on all responses.
- Configure CORS in middleware, not in individual handlers.
- Allowed origins must be explicitly listed. Do not use wildcard `*` in production.

---

## Health Check Endpoint

Every service must expose a health check endpoint outside the `/api/v1/` prefix:

```
GET /health
```

Response (200):
```json
{
  "status": "ok"
}
```

This endpoint must not require authentication and must be excluded from request logging.
