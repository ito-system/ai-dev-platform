# Skill: clone-repo

## Responsibility
Clone an existing GitHub repository to the local environment.

## Input
- `repo_url`: Full HTTPS URL of the repository to clone
- `target_dir` (optional): Local directory name to clone into

## Steps
1. Run: `git clone <repo_url> [target_dir]`
2. Confirm the directory was created and contains the expected files.
3. Output the local path of the cloned repository.

## Output
- Local path of the cloned repository
- Confirmation that clone succeeded

## Notes
- If the clone fails due to auth, prompt the user to check SSH keys or GitHub token.
