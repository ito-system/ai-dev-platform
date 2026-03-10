# Skill: generate-saas-idea

## Responsibility
Generate a validated SaaS product concept — including value proposition, core features, and monetization model — from a target market and problem domain.

---

## Input

- `market` (required): Target market or industry segment (e.g., `"HR teams at SMBs"`, `"freelance designers"`)
- `problem_domain` (required): Area of pain to solve (e.g., `"employee onboarding"`, `"client invoice tracking"`)
- `constraints` (optional): Known constraints to avoid (e.g., `"no hardware"`, `"no regulated industries"`, `"must be buildable by 1-2 engineers"`)

---

## Steps

1. **Identify the core pain point.**
   - State the specific frustration or inefficiency that users in `market` experience within `problem_domain`.
   - Be concrete. Avoid vague problems like "productivity is low."
   - Example output: "HR managers at SMBs manually track onboarding tasks in spreadsheets, causing missed steps and slow ramp times."

2. **Generate a product concept.**
   - Name the product category (e.g., workflow automation, analytics dashboard, communication tool).
   - Describe the solution in one sentence that a non-technical stakeholder can understand.
   - Apply `constraints` to eliminate infeasible directions.

3. **Write the value proposition.**
   - One sentence. Format: `"[Product] helps [market] [achieve outcome] by [mechanism]."`
   - Example: "OnboardIQ helps HR teams at SMBs cut new hire ramp time by 40% by automating task assignment and progress tracking."

4. **Define 3 core features.**
   - Each feature must directly support the value proposition.
   - Each feature must be buildable by a small team (≤2 engineers) within a standard sprint.
   - Format each as: `Feature name — one-sentence description of what it does and why it matters.`

5. **Identify the monetization model.**
   - Select from: `per-seat subscription`, `usage-based`, `flat monthly`, `freemium + upgrade`, `one-time license`.
   - Justify the choice in one sentence based on the market and usage pattern.

6. **Feasibility check.**
   - Confirm the product does NOT require hardware, regulatory approval, or a marketplace with two-sided network effects to deliver the core value proposition.
   - If it fails this check, restart from Step 2 with a revised concept.

---

## Output

A structured product concept document containing:

```
Product Concept
───────────────
Market:             <market>
Problem:            <specific pain point>
Value Proposition:  <one-sentence VP>

Core Features:
  1. <Feature name> — <description>
  2. <Feature name> — <description>
  3. <Feature name> — <description>

Monetization Model: <model> — <one-sentence justification>

Feasibility: PASS / FAIL — <reason if FAIL>
```

This output is consumed by:
- `skills/project/generate-product-name.md` (pass value proposition + market)
- `skills/saas/generate-core-feature.md` (pass core features[1] as the feature to build first)

---

## Validation

- [ ] Value proposition follows the exact format: "[Product] helps [market] [achieve outcome] by [mechanism]."
- [ ] Exactly 3 core features are listed — no more, no fewer.
- [ ] Each core feature description is one sentence and references the value proposition.
- [ ] Monetization model is one of the 5 allowed types.
- [ ] Feasibility check result is explicitly stated as PASS or FAIL.
- [ ] If constraints were provided, confirm none of the features violate them.

---

## Notes

- This skill produces a **concept only** — no code is generated.
- Ideas must be buildable by a team of 1–2 engineers without external APIs that require paid enterprise agreements.
- Do not scope into secondary features. The 3 core features must be the minimum set to deliver the value proposition.
- Idempotent: running this skill with the same inputs produces a logically equivalent concept. If a concept already exists in the project (e.g., a `product-concept.md` file), check its contents first and only regenerate if the market or problem domain has changed.
