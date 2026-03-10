# Refactor Prompt Template

Use this template when prompting Claude to refactor existing code.

---

## Role

Read `prompts/system/senior-engineer.md`.

## Context

Project: <project_name>
Repository: <repo_path>
Target: <file_or_module_to_refactor>

## Refactor goal

**Reason for refactor:** <why_this_needs_to_change>

**Current problem:** <describe_the_current_issue>

**Desired outcome:** <what_good_looks_like_after>

## Constraints

- Do not change external behavior. Tests must still pass.
- Do not expand the scope beyond the target module.
- Follow `shared-context/` conventions for the affected layer.
- Document any interface changes.

## Output expected

- Refactored implementation
- Updated tests (if signatures changed)
- Summary of what changed and why
