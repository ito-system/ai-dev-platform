# Skill: generate-product-name

## Responsibility
Generate and recommend a product name from a set of candidates, evaluated against memorability, domain availability, and audience fit.

---

## Input

- `concept` (required): One-sentence description of what the product does (e.g., `"Automates employee onboarding task assignment for HR teams"`)
- `audience` (required): Target user persona or market segment (e.g., `"HR managers at SMBs"`)
- `style` (optional, default: `"descriptive"`): Naming style — `"descriptive"` | `"abstract"` | `"playful"` | `"compound"`
- `excluded_words` (optional): List of words to avoid (e.g., `["flow", "hub", "suite"]`)

---

## Steps

1. **Understand the naming constraints.**
   - Apply `style` to guide the generation direction:
     - `descriptive`: Name reflects what the product does (e.g., "TaskBoard", "InvoiceNow")
     - `abstract`: Name evokes a feeling or concept (e.g., "Lattice", "Notion")
     - `playful`: Name is light, friendly, human (e.g., "Basecamp", "Loom")
     - `compound`: Two meaningful words combined (e.g., "Salesforce", "Dropbox")
   - Apply `excluded_words` as a hard filter on all candidates.

2. **Generate 5 candidate names.**
   - Each candidate must be:
     - 1–2 words (12 characters maximum, ideally under 10)
     - Easy to pronounce and spell in English
     - Not a common dictionary word used in a misleading way
     - Not already a registered trademark of a well-known product (apply best-effort check)
   - Apply `excluded_words` filter — reject any candidate containing an excluded word.

3. **Evaluate each candidate.**
   - For each of the 5 candidates, score on 4 dimensions (1–3 scale):
     - **Memorability**: Is it easy to recall after hearing it once?
     - **Clarity**: Does it suggest what the product does (relative to style)?
     - **Domain friendliness**: Is it likely available as a `.com` domain?
     - **Audience fit**: Does it resonate with `audience`?
   - Write one sentence of rationale per candidate.

4. **Recommend the strongest name.**
   - Select the candidate with the highest combined score.
   - If two candidates tie, prefer the one with higher memorability.
   - State the recommendation explicitly.
   - Provide a 2-sentence justification covering: why it fits the audience and how it reflects the concept.

5. **Generate usage variants.**
   - For the recommended name, produce:
     - `display_name`: Exact casing as shown to users (e.g., `"OnboardIQ"`)
     - `slug`: kebab-case for URLs (e.g., `"onboard-iq"`)
     - `env_prefix`: SCREAMING_SNAKE_CASE for environment variables (e.g., `"ONBOARD_IQ"`)
     - `go_package`: lowercase single word or underscore-separated for Go package naming (e.g., `"onboardiq"`)

---

## Output

```
Product Name Candidates
───────────────────────
Concept:   <concept>
Audience:  <audience>
Style:     <style>

Candidates:
  1. <Name> — <rationale>
     Scores: Memorability: N/3 | Clarity: N/3 | Domain: N/3 | Audience Fit: N/3

  2. <Name> — <rationale>
     Scores: Memorability: N/3 | Clarity: N/3 | Domain: N/3 | Audience Fit: N/3

  3. <Name> — <rationale>
     Scores: Memorability: N/3 | Clarity: N/3 | Domain: N/3 | Audience Fit: N/3

  4. <Name> — <rationale>
     Scores: Memorability: N/3 | Clarity: N/3 | Domain: N/3 | Audience Fit: N/3

  5. <Name> — <rationale>
     Scores: Memorability: N/3 | Clarity: N/3 | Domain: N/3 | Audience Fit: N/3

Recommendation: <Name>
  Justification: <2 sentences>

Usage Variants:
  display_name: <Name>
  slug:         <name-slug>
  env_prefix:   <NAME_PREFIX>
  go_package:   <namepackage>
```

This output is consumed by:
- `skills/project/generate-brand.md` (pass `display_name` as `product_name`)
- `skills/project/replace-project-name.md` (pass `slug` as `project_name`)

---

## Validation

- [ ] Exactly 5 candidates are listed.
- [ ] No candidate exceeds 12 characters.
- [ ] No candidate contains any word from `excluded_words`.
- [ ] Each candidate has scores on all 4 dimensions.
- [ ] Exactly 1 recommendation is stated with explicit justification (2 sentences).
- [ ] All 4 usage variants (`display_name`, `slug`, `env_prefix`, `go_package`) are generated for the recommended name.
- [ ] `slug` is valid kebab-case (lowercase, hyphens only, no spaces).
- [ ] `env_prefix` is valid SCREAMING_SNAKE_CASE.
- [ ] `go_package` is lowercase with no hyphens.

---

## Notes

- Idempotent: if a `display_name` already exists in the project (e.g., in a config file or README), read it first. Only regenerate if `concept` or `audience` has changed since the name was generated.
- This skill produces a **name recommendation only** — it does not rename any files or replace strings. File renaming is handled by `skills/project/replace-project-name.md`.
- Do not validate domain availability via live DNS lookups. Apply heuristic judgment only.
- Avoid names that are homophones of offensive words in major languages.
