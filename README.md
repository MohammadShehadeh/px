# px

Coding conventions packaged for Claude Code — self-contained `px-*` skills, thin commands, and a reviewer agent, extracted from real production TypeScript/React/Next.js codebases.

## What's inside

```text
skills/       Self-contained skills — each installs standalone into any repo
  px-conventions/
    SKILL.md             Condensed cheat sheet of the house style — triggers on any TS/React work
    references/          The full rules, one topic per file:
      core-principles      How to work: adapt to the repo, plan first, be concise, simplest code that works
      naming               kebab-case files, is/has/should booleans, handle*/on*, no barrels
      typescript           interface vs type, status unions, derive-don't-restate, Array<T>
      components           Named arrow-const components, compound flat exports, flat trees
      hooks-state          Object-returning hooks, one status union, safe context
      forms                RHF + zod logic, Field/FieldGroup markup, control chooser
      nextjs               Server-first, leaf client boundaries, metadata factories
      styling              Tailwind v4 + cn(), semantic tokens, variants, a11y
      ui-composition       shadcn/ui composition API, overlay chooser, a11y invariants
      ui-ux                Reuse over inventing, hierarchy & spacing, responsive, full journey states
      icons                Direct imports, component objects, no sizing classes inside components
      services             Service layer, DTO mapping, zod at boundaries, fetch timeouts/keepalive
      optimistic-ui        useOptimistic + useTransition overlay, debounce + reconcile
      errors               SCREAMING error keys, guard clauses, 'Error in <fn>::' logging
      structure            Feature modules (colocated default), optional packages, no barrels
      testing              Colocated tests on pure logic, match repo toolchain
      review-checklist     Canonical review list — shared by the command and the agent
  px-debug/              Localize top-down, trace references to the root cause, check blast radius
  px-nextjs-page/        Build a page/landing section the house way (+ references/nextjs.md)
  px-feature/            Scaffold a feature module — colocated by default, package when shared (+ references/structure.md)

commands/     Slash commands over the skills
  /plan-task             Restate task as verifiable targets, confirm before coding
  /new-component         Design and build a component: role & states → props API →
                         server/client → build → verify
  /new-feature           Plan and build a feature end to end: user journey & business
                         logic → atomic UI decomposition (page → section → primitive) →
                         state/communication map → home (route vs features/ vs package) →
                         structure → boundaries → no premature abstraction → verify
  /review-conventions    Review the current diff against review-checklist.md

agents/       Subagents
  conventions-reviewer   Ranked violation report for a diff or files (reads review-checklist.md)

templates/    Slim CLAUDE.md template for projects (skills carry the rules; @-imports as fallback)
```

### Naming scheme

- **Skills** are `px-` + noun (`px-conventions`, `px-debug`) — collision-proof when installed standalone, self-contained with their own `references/`.
- **Commands** are verb-first imperatives (`/new-component`, `/review-conventions`). They lean on the skills for the rules; `/new-feature` additionally carries the end-to-end build workflow.
- **One word per concept**: "conventions" (plural) and "review" everywhere — skill `px-conventions`, command `/review-conventions`, agent `conventions-reviewer`, reference `review-checklist.md`.
- `px-nextjs-page` and `px-feature` carry a copy of the one rule file they need so they install alone; each copy is marked with a keep-in-sync note pointing at the original in `px-conventions/references/`.

## Install

**As a plugin** (everything at once — skills, commands, agent):

```text
/plugin marketplace add MohammadShehadeh/px
/plugin install px
```

**Skills only, anywhere** — each skill directory is self-contained (Agent Skills format). Copy any of them into `~/.claude/skills/` (personal) or a repo's `.claude/skills/` (per project):

```text
skills/px-conventions/     →  .claude/skills/px-conventions/
skills/px-debug/           →  .claude/skills/px-debug/
...
```

**Commands & agent** require the plugin install — they resolve the checklist via `${CLAUDE_PLUGIN_ROOT}`. If you copy them out instead, replace that variable with the real path to `skills/px-conventions/`.

**Per project (no skills at all)** — copy `skills/px-conventions/references/` into the repo as `rules/` and use `templates/CLAUDE.md` with its fallback @-imports uncommented.

## The style in one paragraph

Inspect the repo before imposing patterns; named arrow-const components with no default exports and no barrel files; kebab-case filenames; props as `<Component>Props` interfaces; one `status` union per async flow (booleans derived, never stored); **flat component trees** (page → section → primitive, no prop drilling); Tailwind v4 through `cn()` with semantic tokens and the variant → token → wrapper customization ladder; **full composition APIs over raw markup** (Field/FieldGroup forms, Group wrappers, direct icon imports as component objects); server-first Next.js with client boundaries pushed to leaf re-export files; every external system behind a service that maps DTOs and returns `{ ok, data | errorKey }`; optimistic UI on `useOptimistic` + `useTransition`, reconciled by the API response, never a forked copy; **errors are typed SCREAMING_SNAKE codes, never sentences**; guard clauses over conditional mazes; feature modules colocated in the app by default, packages only when shared; plan before coding, simplest code that works, surgical diffs only.

## License

MIT © Mohammad Shehadeh — see [LICENSE.md](LICENSE.md).
