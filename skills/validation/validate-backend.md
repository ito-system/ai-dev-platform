# Skill: validate-backend

## Responsibility
Validate that generated backend code follows clean architecture and API conventions.

## Input
- `backend_path`: Path to the backend source directory

## Checks
1. **Layer separation**: Confirm handler, use-case, and repository layers exist and are separate packages.
2. **API naming**: Confirm endpoints follow `/{resource}/{id}` RESTful conventions.
3. **Error handling**: Confirm domain error types are used (not raw `errors.New`).
4. **Context propagation**: Confirm `context.Context` is the first parameter in all service/use-case functions.
5. **Tests exist**: Confirm at least one `_test.go` file exists per use-case.

## Output
- Pass/Fail for each check
- List of violations with file and line reference
- Overall status: PASS or FAIL

## Notes
- Read `shared-context/go-clean-architecture.md` before running checks.
- A FAIL on any check must block the playbook from completing.
