# Playbook Guide

This guide explains how to design and use playbooks.

---

## What is a playbook?

A playbook is a Markdown file that orchestrates a sequence of skills into a full workflow.

Example: `playbooks/tob-saas/create-tob-saas.md`
→ Generates a complete SaaS product from concept to committed repository.

---

## Playbook structure

Every playbook must follow the four-phase structure:

| Phase | Content |
|---|---|
| Phase 1: Specification | Generate specs using tob-saas/ or project/ skills |
| Phase 2: Implementation | Generate code using development/ skills |
| Phase 3: Validation | Verify artifacts using validation/ skills |
| Phase 4: Commit | Commit changes using github/ skills |

Use `templates/playbook-template.md` as your starting point.

---

## Playbook categories

| Category | Location | Purpose |
|---|---|---|
| SaaS | `playbooks/tob-saas/` | SaaS product workflows |
| General | `playbooks/general/` | Generic project workflows |

---

## Rules for new playbooks

1. Playbooks orchestrate skills. They do not implement logic.
2. Always read relevant `shared-context/` documents before implementation phase.
3. Always end with validation. A FAIL must block completion.
4. Use role prompts from `prompts/system/` at the start.
5. File name must be kebab-case verb-noun: `create-tob-saas.md`.

---

## How to create a new playbook

1. Copy `templates/playbook-template.md`.
2. Define the purpose and inputs.
3. Assign the appropriate role from `prompts/system/`.
4. Build the four phases using skills from `skills/`.
5. Reference shared-context documents in Phase 2.
6. Ensure Phase 3 covers all relevant validation skills.
