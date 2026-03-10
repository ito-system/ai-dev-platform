# Skill: generate-tests

## Responsibility
Generate unit tests for a given function, service, or component.

## Input
- `target`: Name of the function, service, or component to test
- `language`: Target language/framework (e.g., `Go`, `TypeScript/Jest`, `React Testing Library`)
- `test_cases`: List of scenarios to cover (happy path, edge cases, error cases)

## Steps
1. Read `shared-context/testing-guidelines.md`.
2. Generate test file with all specified test cases.
3. Include setup/teardown where appropriate.
4. Mock external dependencies.

## Output
- Test file with all test cases
- List of covered scenarios

## Notes
- Each test must have a clear, descriptive name.
- Tests must be isolated and not depend on each other.
- Coverage must include at least: happy path, one edge case, one error case.
