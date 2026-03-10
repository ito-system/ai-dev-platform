# React Frontend Guidelines

This document defines the frontend architecture standard for React/TypeScript projects.
All frontend code generation skills must follow this guide.

---

## Stack

- React 18+
- TypeScript (strict mode)
- Vite (build tool)
- React Query (server state)
- Zustand (client state, if needed)
- Tailwind CSS (styling)
- React Testing Library + Vitest (testing)

---

## Component rules

- Use functional components only. No class components.
- All props must have explicit TypeScript interfaces.
- Co-locate component files: `ComponentName/index.tsx` + `ComponentName.test.tsx`.
- Keep components small. Extract sub-components when a component exceeds ~150 lines.

---

## Directory structure

```
src/
  components/   # Reusable UI components (no business logic)
  features/     # Feature-scoped components and hooks
  hooks/        # Shared custom hooks
  pages/        # Route-level page components
  api/          # API client functions
  types/        # Shared TypeScript type definitions
  utils/        # Pure utility functions
```

---

## State management

- Server state: React Query (`useQuery`, `useMutation`).
- Local UI state: `useState` / `useReducer`.
- Shared client state: Zustand (avoid unless truly needed).
- Do not use Redux.

---

## API calls

All API calls must go through functions in `src/api/`. Never call `fetch` or `axios` directly from components.

```ts
// src/api/users.ts
export async function getUser(id: string): Promise<User> {
  const res = await apiClient.get(`/users/${id}`);
  return res.data;
}
```

---

## Error and loading states

Every data-fetching component must handle:
- Loading state (skeleton or spinner)
- Error state (error message with retry option)
- Empty state (meaningful empty message)

---

## Testing

- Test behavior, not implementation.
- Use `@testing-library/user-event` for user interactions.
- Mock API calls at the network layer (MSW).
- See `shared-context/testing-guidelines.md`.
