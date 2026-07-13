<!-- Copy of skills/px-conventions/references/nextjs.md so this skill installs standalone — keep in sync. -->

# Next.js App Router

## Server-first

- **Server Components by default; `'use client'` only when needed, pushed to the leaves.** Sections of a page stay server-side; interactivity lives in small leaf wrappers.
- **Centralize client boundaries in re-export files** instead of scattering `'use client'`: a `components/motion.tsx` exporting `MotionDiv`/`MotionSpan` is the single client boundary for animation. (Icons need no boundary — import them directly from the icon library; `px-conventions` skill: `icons` rule.)
- **`'use client'` is justified by:** event handlers and local interactivity, browser APIs, stateful hooks, client-only libraries. **Not by:** rendering data (stay server), animation (`motion.tsx` boundary), icons (server-safe) — and never higher than the leaf that needs it.

## Routing & page composition

- **Route groups `(folder)` organize by domain** without affecting URLs — `(marketing)`, `(dashboard)`, `(content)` — each with its own `layout.tsx`.
- **Pages are thin server components that compose named section components**, each section wrapped in a semantic container:

```tsx
export default function Home() {
  return (
    <>
      <Container gutter="sm" asChild>
        <section><Hero /></section>
      </Container>
      <Container gutter="md" asChild>
        <section><WhatIDo /></section>
      </Container>
    </>
  );
}
```

- **Feature-colocated sub-apps**: a self-contained route folder holds its own `components/`, `lib/`, `store/`, `hooks/`, `constants/`, plus colocated `*.test.ts`. Only cross-feature code goes to top-level `src/components` / `src/lib` / `src/hooks`.
- Use the framework's file conventions (`not-found.tsx`, `global-error.tsx`, `sitemap.ts`, `manifest.json`, `robots.txt`, route handlers) instead of hand-rolling.

## Metadata & SEO

- Root layout defines `metadataBase: new URL(siteConfig.url)` — relative `openGraph`/canonical paths break without it — and `title: { template: '%s - Site Name', default: 'Site Name — tagline' }`; pages set a bare `title` (home omits it and falls back to `default` — with a comment saying why).
- Every page ships full `openGraph` + `twitter` + `alternates.canonical`.
- **Repetitive metadata goes through a factory** — one config object per route, not copy-pasted metadata blocks:

```ts
// lib/page-metadata.ts
interface PageMetaConfig {
  title: string;
  description: string;
  path: string;
}

export const createPageMetadata = ({ title, description, path }: PageMetaConfig): Metadata => ({
  title,
  description,
  alternates: { canonical: path },
  openGraph: { title, description, url: path, images: ['/og.jpg'] },
  twitter: { card: 'summary_large_image', title, description },
});
```

- JSON-LD structured data via a plain `<script>` tag (not `next/script`), built by a sibling factory (`createPageJsonLd(config)`) and injected by the shared page layout — escape `<` against XSS:

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c') }}
/>
```

- `sitemap.ts` is generated from the same typed `src/data/navigation` arrays the nav uses — one source of truth.

## Data & env

- Static content lives in typed `src/data/*` modules (`Array<NavigationItem>`); use `.tsx` data files when entries embed JSX/icons.
- Env vars are typed and validated with zod (`@t3-oss/env-nextjs` style, `src/env.ts`) — never raw `process.env` reads scattered across files.
- Path aliases: `@/*` → `src/*`.
