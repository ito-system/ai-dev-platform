# Experimental Playbooks

This file tracks experimental playbook ideas under development.

> These are NOT production-ready. Do not reference from stable playbooks.

---

## Experiment: auto-review-pr

**Status:** Concept

**Idea:** A playbook that automatically reviews a pull request using the code-reviewer role prompt.

**Steps (draft):**
1. Read the PR diff.
2. Apply `prompts/system/code-reviewer.md`.
3. Generate a structured review with blocking and non-blocking issues.
4. Post the review as a GitHub PR comment.

**Open questions:**
- How to fetch PR diff via MCP?
- Should this block merge or only comment?

---

## Experiment: generate-tob-saas-landing-page

**Status:** Concept

**Idea:** A skill that generates a marketing landing page from a brand and product spec.

**Input:** product name, tagline, core features, brand color palette.

**Output:** Single-page React component with hero, features, and CTA sections.

**Open questions:**
- Should this go in `skills/tob-saas/` or a new `skills/marketing/`?
- What CSS framework assumption?

---

## Experiment: multi-agent-feature-generation

**Status:** Early exploration

**Idea:** Run backend and frontend generation in parallel using two Claude agents.

**Challenge:** Requires coordination of shared data model definitions between agents.

**Next step:** Prototype with a simple feature and evaluate output consistency.
