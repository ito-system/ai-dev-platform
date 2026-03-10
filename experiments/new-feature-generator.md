# Experiment: New Feature Generator v2

**Status:** Draft

**Goal:** Improve the feature generation skill to produce more complete, production-ready output in one pass.

---

## Problem with current approach

`skills/development/generate-backend-api.md` generates handler, use-case, and repository interface.
But it does not generate:
- Repository implementation
- Integration with dependency injection container
- OpenAPI/Swagger documentation

This means developers must manually wire these up.

---

## Proposed improvement

Add an optional `full_implementation` flag to `generate-backend-api.md`.

When `full_implementation: true`:
1. Generate all existing outputs.
2. Generate repository implementation (with `// TODO: implement persistence` stubs).
3. Generate DI wiring snippet.
4. Generate OpenAPI spec snippet.

---

## Draft steps for v2

1. Extend `skills/development/generate-backend-api.md` with the flag.
2. Add a `generate-openapi-spec.md` skill in `skills/development/`.
3. Update `playbooks/tob-saas/add-feature.md` to use `full_implementation: true` by default.
4. Add validation check for OpenAPI spec in `skills/validation/validate-backend.md`.

---

## Risks

- Larger skill = harder to maintain. Consider keeping as two separate skills.
- Repository implementations may be too project-specific to generalize.

---

## Decision needed

Decide: expand existing skill vs. create new skills. Evaluate after prototype.
