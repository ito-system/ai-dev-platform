# Skill: generate-backend-api

## Responsibility
Generate a REST API handler for a given resource following clean architecture.

## Input
- `resource`: The resource name (e.g., `user`, `subscription`)
- `operations`: List of CRUD operations to generate (e.g., `create`, `get`, `list`, `update`, `delete`)
- `language`: Target language (default: `Go`)

## Steps
1. Read `shared-context/go-clean-architecture.md`.
2. Read `shared-context/api-design-guidelines.md`.
3. Generate the handler, use-case, repository interface, and domain model for the resource.
4. Follow the layer separation defined in the clean architecture guide.

## Output
- Domain model struct
- Repository interface
- Use-case implementation
- HTTP handler
- Route registration snippet

## Notes
- Do not implement repository persistence layer (only the interface).
- Use context propagation for all function signatures.
- Error handling must use domain error types.
