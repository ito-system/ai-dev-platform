# Skill: create-repo

## Responsibility
Create a new GitHub repository with the correct settings.

## Input
- `repo_name`: Name of the repository (kebab-case)
- `description`: Short description of the repository
- `visibility`: `public` or `private`

## Steps
1. Use the GitHub MCP tool `mcp__github__create_repository` with the provided inputs.
2. Confirm the repository URL is returned.
3. Output the repository URL to the user.

## Output
- GitHub repository URL
- Confirmation message

## Notes
- Do not initialize with a README unless explicitly requested.
- Default visibility is `private`.
