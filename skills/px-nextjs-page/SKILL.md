---
name: px-nextjs-page
description: Build a Next.js App Router page/landing section the house way — thin server page composing section components, leaf-level client boundaries, metadata via title template and factories. Use when creating pages, landing sections, or route groups.
---

# Next.js Page

Full detail in this skill's [references/nextjs.md](references/nextjs.md).

## Recipe

1. **Page = thin server component** (`export default function`) that only composes named section components:

```tsx
export default function Home() {
  return (
    <>
      <Container gutter="md" asChild>
        <section><Hero /></section>
      </Container>
      <Container gutter="lg" asChild>
        <section><WhatIDo /></section>
      </Container>
    </>
  );
}
```

2. **Sections are server components** in `src/components/` (kebab-case, named arrow-const exports). Content is a typed `as const` array mapped to markup — editing copy never means editing JSX structure. **Keep the tree flat**: page → section → primitive; no pass-through wrappers, no prop drilling past one intermediate.

3. **Interactivity goes to the leaves.** A section needing animation imports `MotionDiv` from the `components/motion.tsx` client boundary — it does not become a client component itself. Icons are imported directly from the project's icon library (they render fine server-side).

4. **Metadata**: root layout owns `metadataBase` and `title: { template: '%s - Site Name', default: '...' }`; the page exports a static `Metadata` object with bare `title`, `description`, full `openGraph` + `twitter`, and `alternates.canonical`. Repeated route families go through a `createPageMetadata(config)` factory plus a JSON-LD factory injected as a plain `<script type="application/ld+json">` with escaped JSON (see [references/nextjs.md](references/nextjs.md) for the snippets).

5. **Routing**: group by domain with `(folder)` route groups, each with its own `layout.tsx`. A self-contained feature route owns its `components/`, `lib/`, `store/`, `hooks/`, colocated tests.

6. Register the route in the typed `src/data/navigation.ts` array — nav and `sitemap.ts` both derive from it.
