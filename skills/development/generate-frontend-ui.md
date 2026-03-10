# Skill: generate-frontend-ui

## Responsibility
Generate a React UI component for a given feature.

## Input
- `component_name`: Name of the component (PascalCase)
- `description`: What the component displays or does
- `data_shape`: Shape of the data the component receives (as TypeScript interface or plain description)

## Steps
1. Read `shared-context/react-frontend-guidelines.md`.
2. Generate the React component with TypeScript.
3. Include prop types, basic styling structure, and loading/error states.
4. Export the component as default.

## Output
- TypeScript React component file
- Props interface definition

## Notes
- Use functional components with hooks only.
- Do not use class components.
- Styling uses CSS Modules or Tailwind (as defined in the project's guidelines).
