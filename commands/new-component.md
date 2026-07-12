---
description: Create a React component following house conventions (kebab-case file, named arrow-const export, interface props, cn styling, flat tree)
argument-hint: <ComponentName> [where / notes]
---

Create a component: $ARGUMENTS

Follow the `code-conventions` skill. Checklist:

- File: kebab-case matching the export (`SubmitButton` → `submit-button.tsx`), placed with its feature (route-colocated `components/` or the package's `components/` dir) — top-level shared components only if genuinely cross-feature.
- Named arrow-const export, no default export. No barrel `index.ts` — consumers import this file directly.
- Props `interface <Component>Props` (no `I` prefix), destructured in the signature; `children: React.ReactNode` if needed.
- **Keep the tree flat**: no pass-through wrappers, no prop drilling past one intermediate — compose via `children`/slot props instead. The component should sit at page → section → primitive depth.
- Server component unless it needs interactivity; if it needs animation, import from the `motion.tsx` boundary instead of adding `'use client'`. Icons import directly from the project's icon library, passed as component objects (never string keys), no sizing classes on icons inside components.
- Tailwind only, merged with `cn()`; semantic tokens, no arbitrary hex, no inline styles; variants as a typed lookup map or CVA per repo convention; `gap-*` not `space-*`; customize via the ladder (variant → layout-only `className` → token → new variant → wrapper).
- Compose existing shadcn/ui primitives from `components/ui` with their full anatomy (Group wrappers, Card Header/Content/Footer, `FieldGroup`/`Field` for form markup) — never rebuild `Separator`/`Skeleton`/`Badge`/`Alert` with raw divs. Dialog/Sheet/Drawer get a Title; Avatar gets a Fallback.
- Content as a typed `as const` array mapped to markup when the component renders a list.
- Handlers `handle*`, callback props `on*`, booleans `is/has/should`.
- Failures render from a typed `errorKey` resolved at render time (``t(`errors.${errorKey}`)`` or the `errorCopy` map) — never a hardcoded error string.
- A11y: focus-visible styles, ARIA label if icon-only, `aria-hidden` on decorative elements.

Keep it minimal — no props or variants nobody asked for.
