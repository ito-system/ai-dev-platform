# Skill: replace-project-name

## Responsibility
Replace all occurrences of a placeholder project name with the real project name across the codebase.

## Input
- `placeholder`: The placeholder string to replace (e.g., `my-app`, `PROJECT_NAME`)
- `project_name`: The real project name to substitute

## Steps
1. Search for all files containing `<placeholder>` using grep or ripgrep.
2. Replace all occurrences in each file.
3. Confirm the number of replacements made.

## Output
- List of files modified
- Total number of replacements

## Notes
- Preserves case variants (e.g., `MyApp`, `my-app`, `MY_APP`) where applicable.
- Do not modify files inside `.git/`.
