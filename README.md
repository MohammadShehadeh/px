# pixel-labs

My coding conventions packaged for Claude Code — rules, skills, commands, and an agent, extracted from real production TypeScript/React/Next.js codebases I've written.

## What's inside

```text
rules/        Canonical convention docs (readable standalone, referenced by everything else)
  core-principles   How to work: plan first, be concise, simplest code that works
  naming            kebab-case files, is/has/should booleans, handle*/on*, no barrels
  typescript        interface vs type, status unions, derive-don't-restate, Array<T>
  components        Named arrow-const components, compound flat exports, flat trees
  hooks-state       Object-returning hooks, one status union, safe context
  forms             RHF + zod logic, Field/FieldGroup markup, control chooser
  nextjs            Server-first, leaf client boundaries, metadata factories
  styling           Tailwind v4 + cn(), semantic tokens, variants, a11y
  ui-composition    shadcn/ui composition API, overlay chooser, a11y invariants
  icons             Direct imports, component objects, no sizing classes inside components
  services          Service layer, DTO mapping, zod at boundaries, fetch timeouts/keepalive
  optimistic-ui     useOptimistic + useTransition overlay, debounce + reconcile
  errors            SCREAMING error keys, guard clauses, 'Error in <fn>::' logging
  structure         Feature packages, wildcard per-file exports, no barrels
  testing           Colocated Vitest on pure logic, formatter/linter gates

skills/       Model-invoked knowledge
  code-conventions     Condensed cheat sheet of all rules — triggers on any TS/React work
  feature-package      How to scaffold a monorepo feature package
  nextjs-page          How to build a page/landing section the house way
  debug                Localize top-down, trace references to the root cause, check blast radius

commands/     Slash commands
  /plan                Restate task as verifiable targets, confirm before coding
  /new-component       Create a component through the conventions checklist
  /new-feature         Scaffold a feature package
  /check-conventions   Review the current diff against the rules

agents/       Subagents
  convention-reviewer  Ranked violation report for a diff or files

templates/    CLAUDE.md template that @-imports the rules into a project
```

## Install

**As a plugin** (everything at once — skills, commands, agent):

```text
/plugin marketplace add <path-or-git-url-to-this-repo>
/plugin install pixel-labs
```

**Personal, everywhere** — copy pieces into `~/.claude/`:

```text
commands/*.md  →  ~/.claude/commands/
agents/*.md    →  ~/.claude/agents/
skills/*       →  ~/.claude/skills/
```

> The skills and agent reference the rules via `${CLAUDE_PLUGIN_ROOT}`, which only exists when installed as a plugin — if you copy them out, copy `rules/` somewhere too and replace those references with its real path.

**Per project (rules only)** — copy `rules/` into the repo and use `templates/CLAUDE.md` as the project's `CLAUDE.md`, or paste the relevant rule files into an existing one.

## The style in one paragraph

Named arrow-const components with no default exports and no barrel files; kebab-case filenames; props as `<Component>Props` interfaces; one `status` union per async flow (booleans derived, never stored); **flat component trees** (page → section → primitive, no prop drilling); Tailwind v4 through `cn()` with semantic tokens and the variant → token → wrapper customization ladder; **full composition APIs over raw markup** (Field/FieldGroup forms, Group wrappers, direct icon imports as component objects); server-first Next.js with client boundaries pushed to leaf re-export files; every external system behind a service that maps DTOs and returns `{ ok, data | errorKey }`; optimistic UI on `useOptimistic` + `useTransition`, reconciled by the API response, never a forked copy; **errors are typed SCREAMING_SNAKE codes, never sentences**; guard clauses over conditional mazes; plan before coding, simplest code that works, surgical diffs only.
