# Testing Guidelines

This document defines testing standards across all projects.
All test generation skills must follow this guide.

---

## Principles

- Tests must be deterministic. No flakiness tolerated.
- Tests must be isolated. No shared mutable state.
- Tests must be fast. No real network or DB calls in unit tests.
- Tests must be readable. The test name must explain what is being tested.

---

## Test naming

Pattern: `Test<Unit>_<Scenario>_<ExpectedResult>`

Examples:
- `TestUserUseCase_GetByID_ReturnsUser`
- `TestUserUseCase_GetByID_ReturnsNotFoundWhenMissing`
- `TestCreateUserHandler_ReturnsBadRequestOnInvalidEmail`

---

## Go testing conventions

```go
func TestUserUseCase_GetByID_ReturnsUser(t *testing.T) {
    // Arrange
    repo := &mockUserRepository{
        user: &domain.User{ID: "123", Email: "a@example.com"},
    }
    uc := NewUserUseCase(repo)

    // Act
    result, err := uc.GetByID(context.Background(), "123")

    // Assert
    require.NoError(t, err)
    assert.Equal(t, "a@example.com", result.Email)
}
```

- Use `testify/assert` and `testify/require`.
- Mock interfaces, not concrete types.
- Use table-driven tests for multiple input scenarios.

---

## Frontend testing conventions

```tsx
it('shows user email after loading', async () => {
  render(<UserProfile userId="123" />);
  expect(await screen.findByText('a@example.com')).toBeInTheDocument();
});
```

- Use React Testing Library. No Enzyme.
- Query by accessible roles/labels, not test IDs.
- Mock API at network level using MSW.

---

## Coverage targets

| Layer | Target |
|---|---|
| Domain / use-cases | 80%+ |
| HTTP handlers | 70%+ |
| Frontend components | 60%+ |
| Utility functions | 90%+ |

---

## What NOT to test

- Framework internals
- Generated code (migrations, protobuf)
- One-liners that delegate directly to a library
