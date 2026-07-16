---
description: Scaffold a service or server boundary the house way — operation contract, zod schema, DTO mapping, narrowed Result with ErrorKey, fetch timeouts
argument-hint: <operation-name> [external system / endpoint]
---

Scaffold a service boundary: $ARGUMENTS

Load the `px-service` skill (and `px-conventions` for structure/testing). Phase 2 is the plan — no code before confirmation.

## 1. Inspect the repo

Find and reuse: `Result<T, K>`, `toErrorKey` / `toCaughtErrorKey`, typed `env.ts`, existing services in the same feature. Match location and naming — do not parallel-invent.

## 2. Define the operation contract

**Present before writing code:**

- Operation name, external system (REST, DB, SDK)
- Success type (`null` or domain shape)
- Payload zod schema → `z.infer` type
- Narrowed `ErrorKey` union — reason keys when known, catch-all only when unknown
- Pure mappers vs I/O shell split (`lib/` tested, `services/` thin)

**Wait for confirmation.**

## 3. Build

- **Schema** colocated — same schema can drive form validation later (`px-form`).
- **Service**: DTO mapping stays inside the file; `AbortSignal.timeout` on every fetch; try/catch with `'Error in <fn>::'`; return `Result`, never throw to UI.
- **Action/handler** (if needed): thin `'use server'` or route handler — validate, delegate, return plain `Result`.
- **Error keys** added to feature's `constants/error-keys.ts`.
- **Tests** on pure mappers in `lib/` — not on fetch wiring.

## 4. Verify

- `Result<T, K>` with `K` narrowed — foreign key is a type error.
- No vendor types escape the service file.
- No `process.env` reads in the service; mappers have tests; typecheck/lint pass.
