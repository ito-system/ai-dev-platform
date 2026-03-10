# Go Clean Architecture Guidelines

This document defines the canonical backend architecture for all Go projects on this platform.
All backend code generation skills and playbooks **must** follow this guide without deviation.

---

## Guiding Principle

Each layer has a single responsibility. Dependencies flow **inward only** — from infrastructure toward domain. The domain layer is pure Go with no external dependencies.

---

## Folder Structure

```
backend/
├── cmd/
│   └── api/
│       └── main.go          # Entry point: wire dependencies, start HTTP server
│
├── internal/
│   ├── domain/              # Pure business entities, value objects, domain errors
│   │   ├── user.go
│   │   ├── organization.go
│   │   └── errors.go
│   │
│   ├── usecase/             # Business logic + repository interfaces
│   │   ├── user_usecase.go
│   │   ├── user_repository.go   # Interface definition lives here
│   │   └── organization_usecase.go
│   │
│   ├── repository/          # Repository implementations (DB, cache, external API)
│   │   ├── user_repository.go
│   │   └── organization_repository.go
│   │
│   ├── handler/             # HTTP handlers — parse input, call use-case, write response
│   │   ├── user_handler.go
│   │   └── organization_handler.go
│   │
│   ├── middleware/          # HTTP middleware (auth, logging, request-id, tenant)
│   │   ├── auth.go
│   │   ├── request_id.go
│   │   └── tenant.go
│   │
│   └── dto/                 # Request/response data transfer objects
│       ├── user_dto.go
│       └── organization_dto.go
│
├── pkg/
│   ├── errors/              # Shared error utilities and wrapping helpers
│   └── logger/              # Logging abstraction (wraps slog)
│
├── database/
│   ├── migrations/          # SQL migration files (numbered, sequential)
│   └── db.go                # DB connection and pool setup
│
└── routes/
    └── routes.go            # Route registration — maps paths to handlers
```

---

## Layer Responsibilities

### domain/
- Defines core business entities as Go structs.
- Defines domain-level sentinel errors (`ErrNotFound`, `ErrUnauthorized`, etc.).
- Contains **no imports** from other internal packages.
- Contains **no database tags**, **no JSON tags** (those belong in DTOs).
- May contain simple value-object methods (e.g., `user.IsActive()`).

```go
// internal/domain/user.go
package domain

import "time"

type User struct {
    ID             string
    OrganizationID string
    Email          string
    Role           string
    CreatedAt      time.Time
    UpdatedAt      time.Time
}
```

```go
// internal/domain/errors.go
package domain

import "errors"

var (
    ErrNotFound      = errors.New("not found")
    ErrAlreadyExists = errors.New("already exists")
    ErrUnauthorized  = errors.New("unauthorized")
    ErrForbidden     = errors.New("forbidden")
    ErrInvalidInput  = errors.New("invalid input")
)
```

---

### usecase/
- Contains all business logic.
- Defines repository interfaces that it requires (dependency inversion).
- Depends only on `domain` and its own interface definitions.
- Never imports `handler`, `repository`, or `database`.
- Never accesses the database directly.

**Interface placement rule:** Repository interfaces are defined in the `usecase` package, not in `repository`. This enforces that the use-case owns the contract.

```go
// internal/usecase/user_repository.go
package usecase

import (
    "context"
    "github.com/your-org/your-app/internal/domain"
)

type UserRepository interface {
    FindByID(ctx context.Context, id string) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    FindByOrganization(ctx context.Context, orgID string) ([]*domain.User, error)
    Save(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id string) error
}
```

```go
// internal/usecase/user_usecase.go
package usecase

import (
    "context"
    "github.com/your-org/your-app/internal/domain"
    "github.com/your-org/your-app/pkg/logger"
)

type UserUseCase struct {
    repo UserRepository
    log  *logger.Logger
}

func NewUserUseCase(repo UserRepository, log *logger.Logger) *UserUseCase {
    return &UserUseCase{repo: repo, log: log}
}

func (u *UserUseCase) GetByID(ctx context.Context, id string) (*domain.User, error) {
    user, err := u.repo.FindByID(ctx, id)
    if err != nil {
        return nil, err
    }
    return user, nil
}
```

---

### repository/
- Implements the interfaces defined in `usecase/`.
- Handles all SQL queries and external data source access.
- Translates database rows into domain entities.
- Must apply `organization_id` filtering in all queries (see Multi-tenancy section).
- Never contains business logic.

```go
// internal/repository/user_repository.go
package repository

import (
    "context"
    "database/sql"
    "errors"
    "github.com/your-org/your-app/internal/domain"
    "github.com/your-org/your-app/pkg/ctxutil"
)

type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) FindByID(ctx context.Context, id string) (*domain.User, error) {
    orgID := ctxutil.OrganizationID(ctx)

    row := r.db.QueryRowContext(ctx,
        `SELECT id, organization_id, email, role, created_at, updated_at
         FROM users WHERE id = $1 AND organization_id = $2`,
        id, orgID,
    )

    var u domain.User
    err := row.Scan(&u.ID, &u.OrganizationID, &u.Email, &u.Role, &u.CreatedAt, &u.UpdatedAt)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, domain.ErrNotFound
    }
    if err != nil {
        return nil, err
    }
    return &u, nil
}
```

---

### handler/
- Parses HTTP requests into DTOs.
- Calls use-case methods.
- Writes HTTP responses using canonical response envelopes.
- Maps domain errors to HTTP status codes.
- Contains **no business logic**.

```go
// internal/handler/user_handler.go
package handler

import (
    "encoding/json"
    "errors"
    "net/http"
    "github.com/your-org/your-app/internal/domain"
    "github.com/your-org/your-app/internal/usecase"
)

type UserHandler struct {
    uc *usecase.UserUseCase
}

func NewUserHandler(uc *usecase.UserUseCase) *UserHandler {
    return &UserHandler{uc: uc}
}

func (h *UserHandler) GetByID(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    user, err := h.uc.GetByID(r.Context(), id)
    if err != nil {
        writeError(w, err)
        return
    }
    writeJSON(w, http.StatusOK, map[string]any{"data": toUserResponse(user)})
}

func writeError(w http.ResponseWriter, err error) {
    switch {
    case errors.Is(err, domain.ErrNotFound):
        writeJSON(w, http.StatusNotFound, errorEnvelope("NOT_FOUND", err.Error()))
    case errors.Is(err, domain.ErrUnauthorized):
        writeJSON(w, http.StatusUnauthorized, errorEnvelope("UNAUTHORIZED", err.Error()))
    case errors.Is(err, domain.ErrForbidden):
        writeJSON(w, http.StatusForbidden, errorEnvelope("FORBIDDEN", err.Error()))
    default:
        writeJSON(w, http.StatusInternalServerError, errorEnvelope("INTERNAL_ERROR", "internal server error"))
    }
}
```

---

### middleware/
- Extracts request metadata (request ID, authenticated user, organization) from headers/tokens.
- Injects values into `context.Context` for downstream use.
- Performs authentication and authorization checks.
- No business logic.

---

### dto/
- Request and response structs with JSON tags.
- Used only at the handler layer.
- Domain entities are never returned directly from handlers — always mapped to DTOs.

```go
// internal/dto/user_dto.go
package dto

import "time"

type CreateUserRequest struct {
    Email string `json:"email" validate:"required,email"`
    Role  string `json:"role"  validate:"required,oneof=admin member viewer"`
}

type UserResponse struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    Role      string    `json:"role"`
    CreatedAt time.Time `json:"created_at"`
}
```

---

## Dependency Rules (Summary)

```
handler  →  usecase interface  →  domain
                ↓
         repository impl  →  domain + DB driver
```

| Layer | May import | Must NOT import |
|---|---|---|
| domain | stdlib only | anything in internal/ |
| usecase | domain | handler, repository, database |
| repository | domain, DB driver | handler, usecase (impl), middleware |
| handler | usecase interfaces, dto | domain (except errors), repository, database |
| middleware | domain (errors), pkg/ | usecase, repository, handler |

---

## Context Propagation

All use-case and repository functions accept `context.Context` as the **first parameter**. This is non-negotiable.

### Context keys

Define context keys in a shared `pkg/ctxutil` package:

```go
// pkg/ctxutil/ctxutil.go
package ctxutil

type contextKey string

const (
    keyRequestID      contextKey = "request_id"
    keyUserID         contextKey = "user_id"
    keyOrganizationID contextKey = "organization_id"
)

func WithRequestID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, keyRequestID, id)
}

func RequestID(ctx context.Context) string {
    v, _ := ctx.Value(keyRequestID).(string)
    return v
}

func WithUserID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, keyUserID, id)
}

func UserID(ctx context.Context) string {
    v, _ := ctx.Value(keyUserID).(string)
    return v
}

func WithOrganizationID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, keyOrganizationID, id)
}

func OrganizationID(ctx context.Context) string {
    v, _ := ctx.Value(keyOrganizationID).(string)
    return v
}
```

### Middleware injection

The `tenant` middleware extracts the organization from the authenticated JWT and injects it into context before any handler runs:

```go
// internal/middleware/tenant.go
func TenantMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        claims := claimsFromContext(r.Context()) // set by auth middleware
        ctx := ctxutil.WithOrganizationID(r.Context(), claims.OrganizationID)
        ctx = ctxutil.WithUserID(ctx, claims.UserID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

---

## Multi-Tenancy Isolation

**Rule:** Every repository query that returns tenant-owned data **must** filter by `organization_id` extracted from the context.

```go
orgID := ctxutil.OrganizationID(ctx)
// orgID must appear in every WHERE clause for tenant-scoped resources
```

Failure to apply this filter is a **critical security defect**. Code review must verify this on every repository method.

Entities that are not tenant-scoped (e.g., global configuration, platform-level records) must be explicitly documented as such in the domain definition.

---

## Validation Rules

- Validate all incoming request fields in the handler layer using struct tags and a validation library (e.g., `go-playground/validator`).
- Return 400 with structured `details` array on validation failure (see `api-design-guidelines.md`).
- Do not re-validate in use-cases. Assume input from handler is valid.
- Domain invariants that cannot be expressed as struct tags are enforced in the use-case with domain errors.

---

## Error Handling Strategy

1. Domain errors are defined as sentinel values in `internal/domain/errors.go`.
2. Repository implementations map driver-level errors (e.g., `sql.ErrNoRows`) to domain errors.
3. Use-cases propagate domain errors upward, wrapping with context using `fmt.Errorf("GetUser: %w", err)`.
4. Handlers map domain errors to HTTP status codes using `errors.Is()`.
5. Unknown errors are logged and returned as 500. The original error is **never** exposed in the response body.

---

## Logging Practices

Use `slog` (Go standard library, 1.21+) for all structured logging.

```go
// pkg/logger/logger.go
package logger

import (
    "context"
    "log/slog"
    "github.com/your-org/your-app/pkg/ctxutil"
)

func FromContext(ctx context.Context) *slog.Logger {
    return slog.Default().With(
        "request_id",      ctxutil.RequestID(ctx),
        "user_id",         ctxutil.UserID(ctx),
        "organization_id", ctxutil.OrganizationID(ctx),
    )
}
```

Usage in use-cases and handlers:

```go
log := logger.FromContext(ctx)
log.Info("user fetched", "user_id", user.ID)
log.Error("failed to fetch user", "error", err)
```

Log level conventions:
- `Info` — normal operations, request completion
- `Warn` — expected exceptional cases (e.g., not found, auth failure)
- `Error` — unexpected failures that require investigation
- Never log sensitive data (passwords, tokens, PII).

---

## Naming Conventions

| Layer | File pattern | Example |
|---|---|---|
| Domain entity | `<entity>.go` | `user.go` |
| Domain errors | `errors.go` | `errors.go` |
| Use-case | `<entity>_usecase.go` | `user_usecase.go` |
| Repository interface | `<entity>_repository.go` (in usecase/) | `user_repository.go` |
| Repository implementation | `<entity>_repository.go` (in repository/) | `user_repository.go` |
| HTTP handler | `<entity>_handler.go` | `user_handler.go` |
| DTO | `<entity>_dto.go` | `user_dto.go` |

---

## Dependency Wiring

All dependencies are wired in `cmd/api/main.go`. No global variables. No `init()` functions for dependency setup.

```go
// cmd/api/main.go
db := database.Connect(cfg.DatabaseURL)

userRepo := repository.NewUserRepository(db)
userUC   := usecase.NewUserUseCase(userRepo, log)
userH    := handler.NewUserHandler(userUC)

r := routes.Setup(userH, ...)
http.ListenAndServe(":8080", r)
```

---

## Testing

- Use-cases: test with mock implementations of repository interfaces. Use `testify/mock` or hand-written mocks.
- Handlers: test with `httptest.NewRecorder()` and mocked use-case interfaces.
- Repositories: test against a real test database (Docker-based, seeded per test).
- No test may hit a live database, external API, or shared state.
- See `shared-context/testing-guidelines.md` for full test standards.
