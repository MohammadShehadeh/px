---
name: feature-package
description: Scaffold or extend a monorepo feature package — workspace packages with wildcard per-file subpath exports and no barrels. Use when creating a new feature package or adding a new area (hooks, actions, services) to an existing one.
---

# Feature Package

How a feature workspace package is built. Detail in `${CLAUDE_PLUGIN_ROOT}/rules/structure.md`.

## Layout

Create only the directories the feature needs — **no `index.ts` barrels anywhere**:

```
packages/feature-<name>/
  src/
    components/   # arrow-const components, kebab-case files
    hooks/        # use-*.ts — logic only, JSX lives in components/; object returns with status + errorKey
    actions/      # 'use server' — return { ok, data | errorKey } shapes
    services/     # external calls, DTO → internal mapping
    types/
    constants/    # error-keys.ts, config.ts (as const + keyof typeof)
    lib/          # pure logic — this is what gets tests, *.test.ts colocated
  package.json
  tsconfig.json
```

## package.json

Wildcard per-file exports so every file is individually importable:

```jsonc
{
  "name": "@feature/<name>",
  "type": "module",
  "sideEffects": false,
  "exports": {
    "./components/*": { "types": "./dist/src/components/*.d.ts", "default": "./src/components/*.tsx" },
    "./hooks/*":      { "types": "./dist/src/hooks/*.d.ts",      "default": "./src/hooks/*.ts" },
    "./actions/*":    { "types": "./dist/src/actions/*.d.ts",    "default": "./src/actions/*.ts" },
    "./constants/*":  { "types": "./dist/src/constants/*.d.ts",  "default": "./src/constants/*.ts" }
  }
}
```

Consumers import the defining file: `import { useUploadFile } from '@feature/upload/hooks/use-upload-file'`.

- Internal deps `workspace:*`; shared-versioned externals (react, next, zod, react-hook-form) via `catalog:`.
- Extend the monorepo's shared tsconfig.

## House patterns inside the package

- UI primitives come from the shared UI package — the feature composes, never redefines base UI.
- No hardcoded user-facing strings: content copy through the shared i18n package with dot-namespaced keys; failures as SCREAMING_SNAKE `ErrorKey` literals from `constants/error-keys.ts`, resolved to copy at render time (see `${CLAUDE_PLUGIN_ROOT}/rules/errors.md`).
- Extensible systems (dynamic forms, pluggable renderers) use the registry pattern: definition objects → Map-backed registry → factory → type-to-component renderer.
- Cross-cutting API concerns via higher-order wrappers: `export const POST = withAuth(handleCreateOrder)`.
- Everything else per the `code-conventions` skill.
