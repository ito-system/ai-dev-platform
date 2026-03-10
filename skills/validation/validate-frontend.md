# Skill: validate-frontend

## Responsibility
Validate that generated frontend code follows React guidelines and builds successfully.

## Input
- `frontend_path`: Path to the frontend source directory

## Checks
1. **TypeScript**: Confirm no TypeScript errors (`tsc --noEmit`).
2. **Component structure**: Confirm all components are functional (no class components).
3. **Props typed**: Confirm all components have explicit prop type definitions.
4. **No unused imports**: Confirm no ESLint `no-unused-vars` violations.
5. **Build succeeds**: Run `npm run build` and confirm exit code 0.

## Output
- Pass/Fail for each check
- List of violations with file and line reference
- Overall status: PASS or FAIL

## Notes
- Read `shared-context/react-frontend-guidelines.md` before running checks.
- A build failure must block the playbook from completing.
