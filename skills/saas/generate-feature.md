# Skill: generate-feature

## Responsibility
Generate a full specification for an additional feature of an existing SaaS product.

## Input
- `product_name`: Name of the SaaS product
- `feature_name`: Name of the feature to specify
- `user_story`: "As a <role>, I want <action> so that <benefit>"

## Steps
1. Expand the user story into detailed acceptance criteria.
2. Define the API endpoints required.
3. Define the data model changes required.
4. List UI components needed.
5. Identify edge cases and error states.

## Output
- Acceptance criteria (checklist)
- API endpoints (method, path, request/response shape)
- Data model changes
- UI components list
- Edge cases

## Notes
- Feature must be independently deployable.
- Must not break existing features.
