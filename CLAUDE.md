# CLAUDE.md — AI Development Platform Rules

This file is the **authoritative development guide** for this repository.
Claude must read and follow these rules when modifying any file in this repository.

---

## Skill design rules

- **One skill = one responsibility.** A skill does exactly one thing.
- Skills must be small and composable.
- Skills must NOT contain multi-step workflows. That is the job of playbooks.
- Skill filenames must use kebab-case and describe the action clearly.
- Every new skill must follow `templates/skill-template.md`.

Prohibited in skills:
- Calling other skills
- Multi-stage logic
- Product-specific code or assumptions

---

## Playbook orchestration rules

- Playbooks orchestrate skills. They do not implement logic themselves.
- Every playbook must follow this sequence:
  1. Read relevant `shared-context/` documents
  2. Execute implementation skills in order
  3. Run validation skills at the end
- Every new playbook must follow `templates/playbook-template.md`.
- Playbooks are placed in `playbooks/saas/` or `playbooks/general/` depending on scope.

---

## Shared-context usage

- `shared-context/` documents are the single source of architectural truth.
- Playbooks must reference the relevant shared-context document before generating code.
- Do not duplicate architectural rules inside individual skills or playbooks.
- When architectural decisions change, update `shared-context/` — not individual files.

---

## Validation workflow

- Every playbook must end with a validation phase.
- Use skills from `skills/validation/` for all checks.
- Validation must cover: backend correctness, frontend quality, database schema, feature completeness.
- Do not mark a playbook complete until validation passes.

---

## Prompt reuse rules

- Role definitions live in `prompts/system/`.
- Reference role prompts at the start of relevant playbooks and skills.
- Do not repeat large role instructions inline — reference the file instead.
- Task-specific prompt templates live in `prompts/` (non-system).

---

## Repository content rules

- This repository contains NO product code.
- All assets must be reusable across multiple projects.
- Product-specific logic must never be committed here.

---

## File placement rules

| Asset type | Location |
|---|---|
| Atomic skill | `skills/<category>/` |
| Workflow playbook | `playbooks/<category>/` |
| Role prompt | `prompts/system/` |
| Task prompt template | `prompts/` |
| Architectural guide | `shared-context/` |
| Document template | `templates/` |
| Config files | `configs/` |
| Platform docs | `docs/` |
| Worked examples | `examples/` |
| Unstable experiments | `experiments/` |

---

## Naming conventions

- All filenames: kebab-case with `.md` extension
- Skill names: verb-noun format (`generate-backend-api.md`, `validate-frontend.md`)
- Playbook names: verb-noun format (`create-saas.md`, `add-feature.md`)
- Shared context: domain-focus format (`go-clean-architecture.md`)

---

## When in doubt

1. Check `templates/` for the right structure.
2. Check `shared-context/` before writing architectural rules.
3. Keep skills small. Move complexity to playbooks.
4. Always validate.
