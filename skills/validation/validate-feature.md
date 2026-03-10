# Skill: validate-feature

## Responsibility
Validate that a generated feature is complete against its specification.

## Input
- `feature_spec_path`: Path to the feature specification file
- `implementation_path`: Path to the implementation directory

## Checks
1. **Acceptance criteria**: Confirm each acceptance criterion in the spec is addressed in the implementation.
2. **API endpoints**: Confirm all specified API endpoints exist in the router/handler files.
3. **Data model**: Confirm all specified data entities exist in the domain model.
4. **Tests**: Confirm tests exist covering the feature's happy path and at least one error case.
5. **No TODO comments**: Confirm no `TODO` or `FIXME` comments remain in the feature code.

## Output
- Pass/Fail for each check
- List of unmet criteria
- Overall status: PASS or FAIL

## Notes
- This skill must always be the last validation run in a feature-level playbook.
