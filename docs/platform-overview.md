# Platform Overview

## Philosophy

The AI development platform treats software development as a **composable workflow problem**.

Instead of writing monolithic prompts, we decompose development into:
- **Skills**: small, reusable, single-purpose instructions
- **Playbooks**: orchestrated workflows that compose skills
- **Shared context**: authoritative architectural documents
- **Validation**: automated quality gates

This makes AI-assisted development **consistent, repeatable, and extensible**.

---

## Why not just write one big prompt?

Large, monolithic prompts:
- Are hard to maintain
- Cannot be reused across projects
- Produce inconsistent results
- Cannot be tested independently

Modular skills:
- Can be updated independently
- Can be reused across multiple playbooks
- Can be tested in isolation
- Produce predictable outputs

---

## The development loop

```
Developer provides inputs
        ↓
Playbook reads shared-context
        ↓
Playbook executes implementation skills
        ↓
Playbook executes validation skills
        ↓
Validation passes → commit and ship
Validation fails  → fix and retry
```

---

## Extending the platform

The platform is designed to grow:

- Add new skills for new code generation patterns
- Add new playbooks for new product types
- Add new shared-context documents for new tech stacks
- Add experiments for ideas not yet ready for production

See:
- `docs/skills-guide.md`
- `docs/playbook-guide.md`
- `docs/architecture.md`

---

## Supported product types

| Type | Primary playbook |
|---|---|
| SaaS | `playbooks/tob-saas/create-tob-saas.md` |
| Feature addition | `playbooks/tob-saas/add-feature.md` |
| General project | `playbooks/general/create-project.md` |
| Any feature | `playbooks/general/implement-feature.md` |
