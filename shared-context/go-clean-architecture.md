# Go Clean Architecture Guidelines

This document defines the backend architecture standard for Go projects in this platform.
All backend code generation skills and playbooks must follow this guide.

---

## Layer structure

```
cmd/
  api/          # Entry point: HTTP server setup, dependency wiring

internal/
  domain/       # Pure business entities and error types (no dependencies)
  usecase/      # Business logic. Depends only on domain and repository interfaces.
  repository/   # Data access interfaces (in usecase pkg) + implementations (here)
  handler/      # HTTP handlers. Depends only on use-cases.
  middleware/   # HTTP middleware (auth, logging, etc.)

pkg/
  errors/       # Shared error types
  logger/       # Logging abstraction
```

---

## Dependency rules

- `domain` has zero dependencies on other internal packages.
- `usecase` depends on `domain` and repository interfaces (defined in `usecase/` package).
- `handler` depends on `usecase` interfaces only.
- `repository` implementations depend on `domain` and external drivers (DB, cache).
- No package may import `handler` or `cmd` from inside `internal/`.

---

## Interface placement

Repository interfaces are defined in the `usecase` package, not in `repository`.

```go
// internal/usecase/user.go
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*domain.User, error)
    Save(ctx context.Context, user *domain.User) error
}
```

---

## Error handling

Define domain errors in `internal/domain/errors.go`:

```go
var (
    ErrNotFound      = errors.New("not found")
    ErrAlreadyExists = errors.New("already exists")
    ErrUnauthorized  = errors.New("unauthorized")
)
```

Handlers map domain errors to HTTP status codes.

---

## Context propagation

All use-case and repository functions must accept `context.Context` as the first parameter.

```go
func (u *UserUseCase) GetByID(ctx context.Context, id string) (*domain.User, error)
```

---

## Naming conventions

| Layer | File pattern | Example |
|---|---|---|
| Domain entity | `<entity>.go` | `user.go` |
| Use-case | `<entity>_usecase.go` | `user_usecase.go` |
| Repository impl | `<entity>_repository.go` | `user_repository.go` |
| HTTP handler | `<entity>_handler.go` | `user_handler.go` |

---

## Testing

- Use-cases are tested with mocked repository interfaces.
- Handlers are tested with httptest and mocked use-cases.
- No test should hit a real database.
- See `shared-context/testing-guidelines.md`.
