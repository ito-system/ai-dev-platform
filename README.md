# ai-dev-platform

AI-driven development platform for automating software workflows using Claude Code and MCP.

## What is this?

This repository is a **reusable AI development platform** — not a product. It contains no application code.
It provides the automation assets needed to build software faster with AI:
skills, playbooks, shared architectural context, prompt definitions, and validation workflows.

Supported product types:
- SaaS applications
- B2B / B2C products
- AI tools and developer tools
- Web applications and internal tools

---

## How it works

### Skills
Skills are small, single-responsibility automation units.
Each skill does exactly one thing.

Example: `skills/development/generate-backend-api.md`
→ Generates a REST API in Go following clean architecture.

### Playbooks
Playbooks orchestrate skills into full development workflows.
They read shared context, call skills in order, and run validation.

Example: `playbooks/tob-saas/create-tob-saas.md`
→ Generates a complete SaaS from scratch.

### Shared Context
Long architectural documents reused across playbooks.
Playbooks reference them to maintain consistency.

Example: `shared-context/go-clean-architecture.md`

### Prompts
Reusable role definitions for Claude.
Prevent repeating large system instructions.

Example: `prompts/system/senior-engineer.md`

### Templates
Standardized document templates for skills, playbooks, and feature specs.

### Validation
Validation skills verify generated code before accepting it.
They are called at the end of every playbook.

---

## Directory overview

| Directory | Purpose |
|---|---|
| `skills/` | Atomic, single-responsibility automation units |
| `playbooks/` | Orchestrated workflows built from skills |
| `prompts/` | Reusable role and task prompt definitions |
| `shared-context/` | Long architectural guidelines reused across playbooks |
| `templates/` | Document templates for creating new assets |
| `configs/` | MCP, GitHub, and coding rule configurations |
| `docs/` | Platform documentation |
| `examples/` | Worked examples of playbooks and skill usage |
| `experiments/` | Experimental workflows and ideas in progress |

---

## Extending the platform

**Add a skill:** Copy `templates/skill-template.md`, fill in the single responsibility, place in the right `skills/` subdirectory.

**Add a playbook:** Copy `templates/playbook-template.md`, define the ordered skill sequence, reference shared-context as needed.

**Add shared context:** Create a new `.md` in `shared-context/` and reference it from relevant playbooks.

**Add an experiment:** Drop it in `experiments/` without affecting production assets.

---

## Guiding principles

- One skill = one responsibility
- Playbooks orchestrate; skills execute
- Validation is mandatory at the end of every playbook
- Shared context is the single source of architectural truth
