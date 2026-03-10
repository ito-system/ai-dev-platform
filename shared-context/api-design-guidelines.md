# API Design Guidelines

This document defines REST API design conventions for all projects.
All backend API generation skills must follow this guide.

---

## URL structure

```
GET    /api/v1/{resources}           # List
GET    /api/v1/{resources}/{id}      # Get one
POST   /api/v1/{resources}           # Create
PUT    /api/v1/{resources}/{id}      # Full update
PATCH  /api/v1/{resources}/{id}      # Partial update
DELETE /api/v1/{resources}/{id}      # Delete
```

- Resources are plural nouns: `/users`, `/subscriptions`, `/invoices`.
- Use hyphens for multi-word resources: `/api/v1/payment-methods`.
- Never use verbs in URLs.

---

## HTTP status codes

| Situation | Code |
|---|---|
| Success (read) | 200 |
| Created | 201 |
| No content (delete) | 204 |
| Bad request (validation) | 400 |
| Unauthorized | 401 |
| Forbidden | 403 |
| Not found | 404 |
| Conflict | 409 |
| Internal error | 500 |

---

## Request / Response format

All requests and responses use JSON.

### Success response
```json
{
  "data": { ... }
}
```

### Error response
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found"
  }
}
```

### List response
```json
{
  "data": [ ... ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

---

## Authentication

- Use Bearer token in `Authorization` header.
- Never send credentials in URL query parameters.

---

## Versioning

- Version via URL path: `/api/v1/`, `/api/v2/`.
- Maintain old versions until clients have migrated.
- Document breaking changes in CHANGELOG.
