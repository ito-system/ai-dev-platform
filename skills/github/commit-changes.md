# Skill: commit-changes

## Responsibility
Stage all modified files and create a git commit with a descriptive message.

## Input
- `message`: Commit message describing the changes
- `files` (optional): Specific files to stage; defaults to all modified files

## Steps
1. Stage files: `git add <files>` or `git add -A` if no files specified.
2. Commit: `git commit -m "<message>"`
3. Output the commit hash.

## Output
- Commit hash
- Confirmation message

## Notes
- Commit message must be concise and in imperative mood (e.g., "Add user authentication endpoint").
- Do not stage `.env` or secrets files.
