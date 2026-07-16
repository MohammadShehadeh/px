---
name: px-conventions
description: House TypeScript/React coding conventions. Use whenever writing or reviewing TypeScript, React, or Next.js code — components, hooks, services, types, styling — so output matches the house style.
---

# Code Conventions

Condensed house style. **Adapt to the repo first** — read existing layout, aliases, UI layer, and toolchain; match established patterns (`core-principles`). Full rules with examples live in this skill's `references/` directory — read the topic file when detail is needed:
`core-principles` · `naming` · `typescript` · `components` · `hooks-state` · `forms` · `nextjs` · `styling` · `ui-composition` · `ui-ux` · `icons` · `services` · `optimistic-ui` · `errors` · `structure` · `testing`

## Cheat sheet

**Process** — inspect the repo before imposing patterns; plan before coding and confirm the approach; be concise; say "I don't know" instead of guessing; simplest code that works; surgical diffs only.

**Naming** — kebab-case filenames (components too); camelCase vars; booleans `is/has/should`; handlers `handle*`, callback props `on*`; name business meaning, not technical accident. `.tsx` iff the file contains JSX.

**No barrels** — never create re-export `index.ts` files; import directly from the file that defines the symbol.

**Components** — named arrow-function `const` exports; **no default exports** except Next.js file conventions. Props: `interface <Component>Props` (no `I` prefix), destructured in the signature. `children: React.ReactNode`. Compound components as flat named exports from one file (`Card`/`CardHeader` — never `Card.Header` statics). Data-driven rendering: typed `as const` arrays + `.map()`, content out of markup.

**Flat trees** — design for the flattest tree possible: page → section → primitive. No pass-through wrapper components; no prop drilling past one intermediate — compose via `children`/slot props, fetch/read data where it's used, lift state only to the lowest common owner. Tracing a prop must never take 3–6 file jumps.

**Hooks** — return an object (never a tuple) exposing `status` plus only the derived flags call-sites read; booleans derived from `status`, never stored. Async state is ONE `status` union (`'idle' | 'loading' | 'success' | 'error'`) in one `useState` — bare union by default, discriminated-with-payload only when data/`errorKey` must be tied to the state; never parallel booleans. Failures carry an `errorKey`, not a message. Context via a `createSafeContext` factory that throws without a provider.

**TypeScript** — `interface` for object shapes, `type` for unions/aliases; string-literal unions or `as const` + `keyof typeof` over enums; derive types (`z.infer`, `ReturnType`, `keyof typeof`) instead of restating; `unknown` over `any`; `Array<T>` over `T[]`; model invalid states out (Draft vs Saved shapes, status unions); JSDoc only where the contract isn't obvious from the signature.

**Styling** — Tailwind v4 only (or the repo's established utility system), no inline styles; `cn()` for all class merging; variants as typed lookup maps or CVA (match the repo); semantic tokens (`bg-muted`) never arbitrary hex or raw palette colors; no manual `dark:` overrides (tokens handle themes); customization ladder: variant → `className` for layout only → token → new variant → wrapper; `gap-*` over `space-*`, `size-*` over `w-/h-`, `truncate`; no manual z-index on overlays; hoist repeated class strings to local consts; a11y classes always (`focus-visible:`, `sr-only`, ARIA on icon-only buttons).

**UI/UX** — reuse existing patterns/tokens/components/layouts over inventing new ones; visual hierarchy tracks importance, spacing follows the design system's scale not eyeballed values, alignment is deliberate; responsive behavior is explicit per breakpoint, not incidental; design the full journey — hover/focus/active/disabled states, loading states, error states, success feedback, and deliberate transitions — not just the default/success path.

**UI components** — base UI is **shadcn/ui** when the project uses it (`components/ui`, added via the shadcn CLI — never hand-written, never another library unless the repo already chose one); use the full composition API (Card = Header/Content/Footer; items inside their Group; `TabsTrigger` inside `TabsList`); existing primitives over custom markup (`Separator`, `Skeleton`, `Badge`, `Alert`); Dialog/Sheet/Drawer always get a Title (`sr-only` ok); `Avatar` always has `AvatarFallback`. Forms markup via `FieldGroup`/`Field` (+ `FieldSet`/`FieldLegend` for groups), `InputGroup` + `InputGroupAddon` for buttons-in-inputs, `ToggleGroup` for 2–7 option sets — never raw divs with `space-y-*`. Icons: import directly from the project's icon library, pass component objects not string keys, no sizing classes on icons inside components.

**Next.js** — Server Components by default, `'use client'` at the leaves; centralize client boundaries in re-export files (`motion.tsx`); route groups `(folder)` by domain; thin pages composing named `<Section />` components; metadata via root title template + `createPageMetadata(config)` factories; typed zod-validated `env.ts`.

**Boundaries** — every external call in a service that returns the shared `Result<T, K>` (`{ ok: true, data } | { ok: false, errorKey, meta? }`, defined once per repo, `K` narrowed to the operation's keys) and never throws to the UI; map vendor DTOs to internal shapes at the boundary; zod validation at the boundary; separate decisions (pure, tested) from actions (thin shells). Optimistic UI (only for predictable, low-stakes, reversible writes — never destructive, money/permission, or server-decided actions; those render `isPending` and wait): `useOptimistic` overlay over the server-owned canonical value, kept alive by a transition (debounce awaited *inside* it); the API response reconciles — automatic revert + `errorKey` toast on failure; the machinery lives in one shared hook (`use-optimistic-value`), never inlined per component; pending writes flush on unmount (`keepalive: true` around tab close). Every service fetch sets `AbortSignal.timeout`.

**Errors** — errors are **codes, not sentences**: services return a SCREAMING_SNAKE `errorKey` literal from the string-literal `ErrorKey` union — shared keys are transport/session only (`NETWORK`, `TIMEOUT`, `UNAUTHORIZED`, `RATE_LIMITED`), everything else feature-prefixed, **named by reason when known** (`INVOICE_NOT_FOUND`), the operation catch-all (`INVOICE_FETCH_FAILED`) only when the reason is unknown — no `_FAILED` on reason keys. `Result<T, K>` narrows per operation so a foreign key is a type error; map once via shared helpers (`toErrorKey`: HTTP status → shared keys; `toCaughtErrorKey`: `TimeoutError` → `TIMEOUT` vs `NETWORK`); optional `meta` for interpolation values only. The frontend resolves copy at render time (``t(`errors.${errorKey}`, meta)`` or an exhaustive `Record<ErrorKey, string>`). Guard clauses + early returns, no `else` mazes; try/catch at I/O with `console.error('Error in <fn>::', error)`; raw error details stop at the log.

**Structure** — feature modules colocated in the app by default (route folder or `features/<name>/`); shared package only for 2+ consumers; responsibility-based dirs (`components/`, `hooks/`, `actions/`, `services/`, `types/`, `constants/`, `lib/`); wildcard per-file package exports when extracted; no barrels; Vitest (or repo test runner) on pure logic in `lib/`.
