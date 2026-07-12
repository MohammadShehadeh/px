---
description: Scaffold a monorepo feature package with wildcard per-file exports and no barrels
argument-hint: <feature-name> [what it does]
---

Scaffold a feature package: $ARGUMENTS

Use the `feature-package` skill as the blueprint. Steps:

1. Confirm the target monorepo and package scope by reading an existing sibling package first — mirror its tsconfig extends, catalog usage, and script names exactly.
2. Create `packages/feature-<name>/` with ONLY the directories this feature actually needs (`components/`, `hooks/`, `actions/`, `services/`, `types/`, `constants/`, `lib/`). **No `index.ts` barrels anywhere.**
3. Write `package.json`: scoped name, `"type": "module"`, `"sideEffects": false`, wildcard per-file subpath `exports` (`"./hooks/*"`, `"./components/*"`) with `types`/`default` pairs. Internal deps `workspace:*`, shared externals `catalog:`.
4. Implement per the `code-conventions` skill: status-union hooks with `errorKey`, service-layer boundary with DTO mapping and `Result` returns, guard clauses, `constants/error-keys.ts` for failures, i18n keys not hardcoded copy.
5. Pure logic goes in `lib/` with colocated `*.test.ts` Vitest tests.

Present the plan (dirs + exports map) before writing files.
