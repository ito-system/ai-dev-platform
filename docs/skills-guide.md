# Skills Guide

This guide explains how to create and use skills.

---

## What is a skill?

A skill is a Markdown file that instructs Claude to perform a single, well-defined task.

Example: `skills/development/generate-backend-api.md`
→ Given a resource name and operations, generate a clean architecture REST API in Go.

---

## Skill structure

Every skill must have:

| Section | Purpose |
|---|---|
| `## Responsibility` | One sentence describing the skill's purpose |
| `## Input` | Required and optional inputs |
| `## Steps` | Numbered, concrete steps Claude executes |
| `## Output` | What the skill produces |
| `## Notes` | Constraints, shared-context references |

Use `templates/skill-template.md` as your starting point.

---

## Skill categories

| Category | Location | Purpose |
|---|---|---|
| GitHub | `skills/github/` | Repository and commit operations |
| Project | `skills/project/` | Project-level setup tasks |
| SaaS | `skills/saas/` | SaaS ideation and specification |
| Development | `skills/development/` | Code generation (API, UI, DB, tests) |
| Validation | `skills/validation/` | Quality and consistency checks |

---

## Rules for new skills

1. One skill = one responsibility.
2. Skills must not call other skills.
3. Skills must not contain multi-step workflows.
4. Reference `shared-context/` documents where architectural conventions apply.
5. File name must be kebab-case verb-noun: `generate-backend-api.md`.

---

## How to create a new skill

1. Copy `templates/skill-template.md`.
2. Fill in all sections.
3. Place in the correct `skills/<category>/` subdirectory.
4. Reference the skill from a playbook or example.
