---
name: px-feature
description: Scaffold or extend a feature module — colocated route/feature folder by default, optional shared package when multi-consumer. Use when creating a new feature or adding an area (hooks, actions, services) to an existing one.
---

# Feature Module

How a feature is structured and where it lives. Full detail in this skill's [references/structure.md](references/structure.md). Load `px-conventions` for everything inside the module (components, hooks, boundaries, errors).

## 1. Pick the feature's home

Inspect the repo first — match its existing pattern:

| Situation | Home |
| --- | --- |
| **Default** — single app, one consumer | **Feature-colocated** — route folder under `app/` *or* `src/features/<name>/`, whichever the repo already uses |
| Shared by 2+ apps or versioned independently | **Package** — npm package or monorepo workspace member |

Never manufacture a package for one consumer.

## 2. Layout (create only what you need)

No `index.ts` barrels anywhere:

```
<feature-home>/
  components/   # arrow-const components, kebab-case files
  hooks/        # use-*.ts — logic only; object returns with status + errorKey
  actions/      # 'use server' — return { ok, data | errorKey }
  services/     # external calls, DTO → internal mapping
  types/
  constants/    # error-keys.ts, config.ts (as const + keyof typeof)
  lib/          # pure logic — colocate *.test.ts here
```

Add a directory only when a file exists for it.

## 3. Imports

Consumers import the **defining file** via the project's path alias:

```ts
import { useUploadFile } from '@/features/upload/hooks/use-upload-file';
import { Dropzone } from '@/features/upload/components/dropzone';
```

## 4. Package boundary (only when extracted)

When the feature becomes a shared package, expose wildcard per-file subpath exports:

```jsonc
{
  "name": "@scope/upload",
  "type": "module",
  "sideEffects": false,
  "exports": {
    "./components/*": { "types": "./dist/components/*.d.ts", "default": "./src/components/*.tsx" },
    "./hooks/*":      { "types": "./dist/hooks/*.d.ts",      "default": "./src/hooks/*.ts" },
    "./actions/*":    { "types": "./dist/actions/*.d.ts",    "default": "./src/actions/*.ts" }
  }
}
```

Follow the repo's package naming, build output paths, and workspace dependency conventions — do not introduce a monorepo layout into a single-package app.

## 5. Patterns inside the module

- Compose the project's shared UI layer — never redefine base primitives.
- User-facing copy via the project's i18n (or an exhaustive key→string map); failures as SCREAMING_SNAKE `ErrorKey` literals from `constants/error-keys.ts` (`px-conventions`: `errors` rule).
- Extensible systems (3+ variants): registry pattern — definitions → Map registry → factory → renderer. Two cases: lookup map or `switch`.
- Cross-cutting API checks: higher-order wrappers (`export const POST = withAuth(handleCreateOrder)`) when the repo already uses that pattern.
- Everything else per `px-conventions`.
