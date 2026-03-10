# Bugfix Prompt Template

Use this template when prompting Claude to fix a bug.

---

## Role

Read `prompts/system/senior-engineer.md`.

## Context

Project: <project_name>
Repository: <repo_path>

## Bug description

**Title:** <bug_title>

**Steps to reproduce:**
1. <step_1>
2. <step_2>
3. <step_3>

**Expected behavior:** <what_should_happen>

**Actual behavior:** <what_actually_happens>

**Error message / stack trace (if any):**
```
<paste_error_here>
```

## Constraints

- Fix only the root cause. Do not refactor unrelated code.
- Add a regression test that would have caught this bug.
- Confirm the fix does not break existing tests.

## Output expected

- Root cause analysis
- Fix implementation
- Regression test
- Confirmation that all tests pass
