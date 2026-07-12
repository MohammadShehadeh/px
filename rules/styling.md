# Styling

- **Tailwind v4 classes for everything; no inline `style` attributes.** CSS-first config (`@import "tailwindcss"`, `@theme inline`) — no `tailwind.config` file in v4 projects.
- **`cn()` (`twMerge(clsx(...))`) is the universal class-merge utility.** Every conditional or merged `className` goes through it — never manual ternaries inside template strings:

```tsx
className={cn('cursor-grab p-1', isDragging ? 'bg-primary/10 cursor-grabbing' : 'hover:bg-muted')}
```

- **Variants are typed lookup maps or CVA** — pick what the repo already uses:
  - Lookup map keyed by a union (lightweight, no dependency):

    ```tsx
    export const headingVariants = { H1: 'text-3xl font-bold', H2: 'text-xl font-bold', P: 'tracking-wider' };
    export type HeadingVariant = keyof typeof headingVariants;
    ```

  - CVA with `defaultVariants` for shadcn-style primitives (`buttonVariants`), merged as `React.ComponentProps<'button'> & VariantProps<typeof buttonVariants>` with an `asChild` Slot escape hatch.

## Customizing a component — escalation ladder

Prefer these in order; stop at the first that works:

1. **Built-in variant** — `<Button variant="outline" size="sm">`.
2. **`className` for layout only** — `max-w-md`, `mx-auto`, `mt-4`. Never use `className` to recolor or re-typeset a component.
3. **Semantic tokens / CSS variables** — add the token to the theme, not a one-off class.
4. **New variant in the component source** — extend the CVA/lookup map.
5. **Wrapper component** — compose primitives into a higher-level component.

```tsx
// Bad — recoloring via className
<Card className="bg-blue-100 text-blue-900 font-bold">...</Card>

// Good — className is layout only
<Card className="mx-auto max-w-md">...</Card>
```

## Tokens and colors

- **Semantic design tokens over raw values**: `text-primary`, `bg-muted`, `text-muted-foreground` — not `bg-[#0067A6]` or `text-green-500`. Arbitrary hex and raw palette colors are a smell; add a token or use a `Badge` variant for status indicators.
- **No manual `dark:` color overrides.** Tokens handle both themes via CSS variables: `bg-background text-foreground`, not `bg-white dark:bg-gray-950`. Dark mode via a `.dark` class + `@custom-variant dark`, tokens defined under `:root` / `.dark`.

## Utility hygiene

- `gap-*` over `space-x-*` / `space-y-*`: `space-y-4` → `flex flex-col gap-4`.
- `size-*` over `w-* h-*` when equal: `size-10`, not `w-10 h-10`.
- `truncate` over `overflow-hidden text-ellipsis whitespace-nowrap`.
- **No manual `z-index` on overlay components** — `Dialog`, `Sheet`, `Drawer`, `Popover`, `Tooltip` manage their own stacking; never add `z-50`.
- Repeated class strings inside a component are hoisted to local `const` style tokens (`const buttonBaseStyle = 'flex h-11 items-center gap-2 rounded'`).
- Mobile-first responsive prefixes (`md:`, `lg:`); logical/RTL-aware utilities (`me-2`, `ps-4`).

## Accessibility

Not optional: `focus-visible:ring-2`, `sr-only`, `aria-hidden` on decorative elements, ARIA labels on icon-only triggers, `reducedMotion="user"` on animation configs.

## Primitives over custom markup

**The base UI layer is shadcn/ui** (`components/ui` or the monorepo UI package, added via the shadcn CLI). Features never define base UI primitives — they compose the shadcn ones. See [ui-composition.md](ui-composition.md) for composition rules.
