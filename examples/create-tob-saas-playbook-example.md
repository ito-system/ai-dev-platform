# Example: Running create-tob-saas Playbook

This example shows a full run of `playbooks/tob-saas/create-tob-saas.md`.

---

## Inputs provided

```
market: "HR managers at companies with 50-500 employees"
problem_domain: "employee onboarding"
```

---

## Phase 1: Ideation output

**Skill: generate-tob-saas-idea**

- Product: OnboardFlow
- Value proposition: "Automate employee onboarding so HR teams save 10 hours per new hire."
- Core features:
  1. Onboarding checklist builder
  2. Automated task assignment to new hire and manager
  3. Progress tracking dashboard
- Monetization: SaaS subscription, per-seat pricing

**Skill: generate-product-name**

Recommended: **OnboardFlow**

**Skill: generate-brand**

- Tagline: "Onboarding, automated."
- Tone: Professional, Efficient, Friendly
- Colors: Deep blue (trust), Green (progress), White (clarity)

---

## Phase 2: Core feature spec output

**Skill: generate-core-feature**

Feature: Onboarding Checklist

User story: As an HR manager, I want to create an onboarding checklist so that new hires complete all required steps.

Acceptance criteria:
- [ ] HR manager can create a checklist with multiple tasks
- [ ] Each task has a name, description, and due-day offset
- [ ] Checklist can be assigned to a new hire
- [ ] New hire sees their assigned checklist on login

API contract:
- `POST /api/v1/checklists` — create checklist
- `GET /api/v1/checklists/{id}` — get checklist
- `POST /api/v1/checklists/{id}/assign` — assign to employee

Data entities: `Checklist`, `ChecklistItem`, `ChecklistAssignment`

---

## Phase 3: Validation output

- validate-database: PASS
- validate-backend: PASS
- validate-frontend: PASS
- validate-feature: PASS

---

## Final output

- GitHub repository: `https://github.com/ito-system/onboardflow`
- Initial commit: `Initial commit: OnboardFlow SaaS scaffold`
