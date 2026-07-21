---
name: px-service
description: Scaffold a service or server boundary the house way — one shared http client, zod-validated payloads, DTO mapping, narrowed Result with ErrorKey, no throws to UI. Use when adding API clients, server actions, route handlers, or external integrations.
---

# Service & Boundary

How to wrap external chaos behind a typed boundary. Full rules in [references/services.md](references/services.md) and [references/errors.md](references/errors.md). Load `px-conventions` for naming, structure, and testing.

## 1. Inspect the repo

Before creating files, find what already exists:

- `Result<T, K>` type — reuse it; define once if missing (`types/result.ts`)
- Shared `http` client (`lib/http.ts`) — reuse it; the service never calls `fetch` directly. Define once if missing (owns base URL, auth headers, JSON, timeout, status→`errorKey` mapping)
- `toErrorKey` / `toCaughtErrorKey` helpers — reuse from `lib/error-keys.ts` (the client uses these; a raw-SDK service that can't go through `http` uses them directly)
- Typed `env.ts` — never read `process.env` in the service
- Existing services in the same feature — match file location, naming, and how they call `http`

## 2. Define the operation contract

**Present this before writing code:**

| Item | Example |
| --- | --- |
| Operation name | `submitContactMessage` |
| Success shape | `null` or domain type `Invoice` |
| Payload | zod schema → `z.infer` type |
| Error keys (narrowed) | `'CONTACT_SUBMIT_FAILED' \| SharedErrorKey` + reason keys like `'CONTACT_RATE_LIMITED'` when known |
| External system | REST endpoint, DB, vendor SDK |

Add new keys to the feature's `constants/error-keys.ts`. Name **reason when known** (`INVOICE_NOT_FOUND` on 404); operation catch-all only when reason is unknown.

## 3. Separate decisions from actions

| Layer | Lives in | Tested |
| --- | --- | --- |
| Pure mapping / validation logic | `lib/` — `toInvoice`, `buildQuery` | Yes — colocated `*.test.ts` |
| I/O shell | `services/` or `actions/` | Integration/manual; logic stays in `lib/` |

Vendor DTO shapes never leave the service file — map to internal domain types inside.

## 4. Service function template

The service goes through the shared `http` client (`lib/http.ts`) — no `fetch`, no headers, no `try/catch`, no timeout in the service. It names the endpoint, narrows the keys, and maps the DTO:

```ts
// services/contact.ts — write with no body payload back
import type { Result } from '@/types/result';
import { http } from '@/lib/http';
import { type ContactFormPayload } from '../lib/contact-schema';

export const submitContactMessage = (
  payload: ContactFormPayload,
): Promise<Result<null, ContactErrorKey>> =>
  http.post<null, ContactErrorKey>('/contact', payload, { fallbackKey: 'CONTACT_SUBMIT_FAILED' });

// services/invoice.ts — read + DTO map: http returns the raw DTO, the service maps it
export const fetchInvoice = async (id: string): Promise<Result<Invoice, InvoiceErrorKey>> => {
  const res = await http.get<VendorInvoiceDto, InvoiceErrorKey>(`/invoices/${id}`, {
    fallbackKey: 'INVOICE_FETCH_FAILED',
    notFoundKey: 'INVOICE_NOT_FOUND',
  });
  return res.ok ? { ok: true, data: toInvoice(res.data) } : res;
};
```

- **Never throw to the UI** — return `{ ok: false, errorKey }`. The client already does this; a service that maps a DTO just forwards `res` on failure.
- **Timeout, auth headers, and status→`errorKey` mapping live in the client** — pass `signal` for caller cancellation, `notFoundKey` for a 404 reason key.
- **Writes around tab close**: pass `keepalive: true` in the options on fire-and-forget flushes only — never on reads.
- **Untrusted boundary**: `schema.safeParse(res.data)` in the service before mapping; the client stays generic.
- A service that must call a **raw vendor SDK** (not HTTP) keeps its own `try/catch` with `toCaughtErrorKey` — the `http` client is for fetch.

## 5. Schema at the boundary

Colocate zod schema with the operation; one schema drives service validation and form validation:

```ts
export const contactFormSchema = z.object({
  email: z.string().email(),
  message: z.string().min(10),
});
export type ContactFormPayload = z.infer<typeof contactFormSchema>;
```

Validate inbound payloads at the top of the service (or in the action before delegating).

## 6. Server action / route handler shell

**Server action** (`actions/`): thin `'use server'` wrapper — auth check, validate, call service, return `Result` plain object.

```ts
'use server';

export const submitContact = async (payload: ContactFormPayload): Promise<Result<null, ContactErrorKey>> => {
  const parsed = contactFormSchema.safeParse(payload);
  if (!parsed.success) return { ok: false, errorKey: 'CONTACT_VALIDATION_FAILED' };
  return submitContactMessage(parsed.data);
};
```

**Route handler**: use the repo's higher-order wrapper when one exists (`export const POST = withAuth(handleCreateOrder)`); otherwise guard clauses at the top, delegate to service.

## 7. File layout

Colocate with the feature:

```
services/contact.ts           # I/O + DTO mapping
actions/submit-contact.ts     # 'use server' thin shell (if needed)
lib/contact-schema.ts         # zod + mappers — tested
constants/error-keys.ts       # ContactErrorKey union
types/result.ts               # shared Result (repo-level, once)
lib/http.ts                   # shared fetch client (repo-level, once)
lib/error-keys.ts             # toErrorKey, toCaughtErrorKey (repo-level, once)
```

Kebab-case filenames, named exports, no barrel `index.ts`.

## 8. Verify

- Service returns `Result<T, K>` with `K` narrowed to this operation's keys — foreign key is a type error at call sites.
- No raw `fetch` / headers / `AbortSignal.timeout` in the service — it goes through the shared `http` client (raw-SDK services excepted).
- No vendor DTO types imported outside the service file.
- No thrown errors cross the UI boundary; `'Error in <fn>::'` logging on every catch.
- Pure mappers in `lib/` have tests; typecheck and lint pass.
- Final test: would a senior engineer call this overcomplicated?
