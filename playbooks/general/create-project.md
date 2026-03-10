# Playbook: create-project

## Purpose
Create a new software project repository with standard structure.

## Role
Read `prompts/system/architect-role.md` before starting.

## Inputs
- `project_name`: Name of the project
- `description`: One-sentence description
- `visibility`: `public` or `private`

---

## Steps

1. Execute skill: `skills/project/generate-product-name.md` (if no name provided)
2. Execute skill: `skills/github/create-repo.md`
   - Input: project name, description, visibility
3. Execute skill: `skills/github/clone-repo.md`
4. Execute skill: `skills/project/replace-project-name.md`
   - Replace placeholder with real project name
5. Execute skill: `skills/github/commit-changes.md`
   - Message: `"Initial project setup"`

---

## Output
- GitHub repository URL
- Local project path
- Confirmation of initial commit
