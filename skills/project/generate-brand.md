# Skill: generate-brand

## Responsibility
Generate a complete brand identity definition — tagline, tone of voice, color palette, and positioning statement — that is consistent with the product's value proposition and target audience.

---

## Input

- `product_name` (required): Display name of the product (e.g., `"OnboardIQ"`)
- `concept` (required): One-sentence description of what the product does
- `audience` (required): Target user persona or market segment (e.g., `"HR managers at SMBs"`)
- `value_proposition` (required): One-sentence VP from `generate-tob-saas-idea` output
- `tone_preference` (optional): Preferred tone direction — `"professional"` | `"friendly"` | `"bold"` | `"minimal"` (default: derived from audience)

---

## Steps

1. **Derive tone of voice.**
   - If `tone_preference` is provided, use it as the anchor.
   - If not, derive from `audience`:
     - Enterprise / regulated industries → `"professional"`
     - SMBs / teams → `"friendly"`
     - Developers / technical users → `"minimal"`
     - Consumer / B2C → `"friendly"` or `"bold"`
   - Select exactly 3 tone adjectives that define how the brand communicates.
   - Each adjective must be distinct (not synonyms).
   - For each adjective, write one example sentence showing the brand voice in practice.

2. **Write the tagline.**
   - Maximum 8 words.
   - Must reflect the value proposition without copying it verbatim.
   - Must match the derived tone of voice.
   - Generate 3 tagline options, then select the strongest.
   - Justification: one sentence explaining why the selected tagline is strongest.

3. **Define the color palette.**
   - Define 3 colors: `primary`, `secondary`, `accent`.
   - For each color:
     - Provide a descriptive name (e.g., "Trust Blue", "Warm Slate")
     - Provide a hex code
     - State the purpose (e.g., "Primary CTA buttons, active navigation items")
   - Colors must align with the tone of voice:
     - `"professional"` → Blues, grays, neutral palettes
     - `"friendly"` → Warm tones, accessible greens, soft blues
     - `"bold"` → High-contrast, saturated primaries
     - `"minimal"` → Near-monochrome with a single accent
   - Verify contrast ratio guidance: primary text on background must meet WCAG AA (4.5:1).

4. **Write the brand positioning statement.**
   - Exactly 2 sentences.
   - Sentence 1: Who the product is for and what it does.
   - Sentence 2: How it is different from generic alternatives.
   - Must not contain marketing clichés ("world-class", "revolutionary", "game-changer").

5. **Define copy tone rules.**
   - List 3 "do" rules (what the brand always does in copy).
   - List 3 "don't" rules (what the brand never does in copy).
   - These rules must be specific enough to distinguish this brand from a generic SaaS product.

---

## Output

```
Brand Identity
──────────────
Product:   <product_name>
Audience:  <audience>

Tone of Voice:
  Adjectives: <adjective 1>, <adjective 2>, <adjective 3>
  Examples:
    <adjective 1>: "<example sentence in brand voice>"
    <adjective 2>: "<example sentence in brand voice>"
    <adjective 3>: "<example sentence in brand voice>"

Tagline:
  Options:
    1. "<tagline option 1>"
    2. "<tagline option 2>"
    3. "<tagline option 3>"
  Selected: "<selected tagline>"
  Justification: <one sentence>

Color Palette:
  Primary:   <name> — #<hex> — <purpose>
  Secondary: <name> — #<hex> — <purpose>
  Accent:    <name> — #<hex> — <purpose>
  WCAG AA guidance: <pass/guidance note>

Positioning Statement:
  <Sentence 1: who it's for and what it does.>
  <Sentence 2: how it differs from alternatives.>

Copy Tone Rules:
  DO:
    - <rule 1>
    - <rule 2>
    - <rule 3>
  DON'T:
    - <rule 1>
    - <rule 2>
    - <rule 3>
```

---

## Validation

- [ ] Exactly 3 tone adjectives are defined, each with an example sentence.
- [ ] Tagline is 8 words or fewer.
- [ ] 3 tagline options generated; 1 selected with justification.
- [ ] Color palette defines exactly 3 colors (primary, secondary, accent).
- [ ] Each color has a name, hex code, and purpose statement.
- [ ] Color palette is consistent with the tone of voice.
- [ ] WCAG AA guidance note is present.
- [ ] Positioning statement is exactly 2 sentences.
- [ ] Positioning statement contains no clichés ("world-class", "revolutionary", "game-changer", "best-in-class").
- [ ] Copy tone rules: exactly 3 DOs and 3 DON'Ts.
- [ ] DO and DON'T rules are specific to this brand, not generic advice.

---

## Notes

- Idempotent: if a brand document already exists for this product, read it first. Only regenerate sections where the inputs have changed. Do not overwrite an approved brand without explicit instruction.
- This skill produces a **brand definition document only** — no code or UI assets are generated.
- The color palette is a direction, not a final design spec. A designer must validate hex codes against actual UI components.
- Tone of voice adjectives and copy rules feed directly into `skills/development/generate-frontend-ui.md` to ensure UI copy matches brand voice.
