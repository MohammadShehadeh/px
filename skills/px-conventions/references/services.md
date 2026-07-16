# Services & Boundaries

**Put boundaries around external chaos.** External systems get an adapter; their shapes never become the language of the codebase.

- Do not let raw database rows leak into UI logic.
- Do not let HTTP response shapes leak into domain logic.
- Do not let framework-specific request objects leak into business services.
- Do not let environment variable parsing happen randomly across files — one typed, zod-validated `env.ts`.
- Do not let Stripe, Slack, GitHub, Salesforce, or any other external system become the language of your entire codebase.

## The service layer

- Every external call lives in a **service function** (`services/`, `actions/`) that returns a **normalized, UI-shaped `Result` and never throws to the UI**. Failures are error keys (see [errors.md](errors.md)):

```ts
// types/result.ts — defined once per repo, imported everywhere; K narrows to the keys
// the operation can actually produce (see errors.md)
export type Result<T, K extends ErrorKey = ErrorKey> =
  | { ok: true; data: T }
  | { ok: false; errorKey: K; meta?: Record<string, string | number> };

// services/contact.ts — paths use the project's alias convention
import type { Result } from '@/types/result';
import { toCaughtErrorKey, toErrorKey } from '@/lib/error-keys';

export const submitContactMessage = async (payload: ContactFormPayload): Promise<Result<null, ContactErrorKey>> => {
  try {
    const response = await fetch(`${env.API_BASE_URL}/contact`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
      signal: AbortSignal.timeout(10_000),
    });
    if (!response.ok) return { ok: false, errorKey: toErrorKey(response, 'CONTACT_SUBMIT_FAILED') };
    return { ok: true, data: null };
  } catch (error) {
    console.error('Error in submitContactMessage::', error);
    return { ok: false, errorKey: toCaughtErrorKey(error) };
  }
};
```

- **Map external DTOs to internal shapes at the boundary.** The vendor response is consumed inside the service; only the domain shape leaves it:

```ts
// services/billing.ts — vendor snake_case never escapes this file
const toSubscription = (dto: VendorSubscriptionDto): Subscription => ({
  id: dto.subscription_id,
  status: dto.state === 'ACTIVE' ? 'active' : 'canceled',
  renewsAt: new Date(dto.next_billing_time),
});
```

- **Validate at the boundary with zod.** Schemas co-located with the service; payload types via `z.infer`. The same schema drives form validation:

```ts
export const contactFormSchema = z.object({
  email: z.string().email(),
  message: z.string().min(10),
});
export type ContactFormPayload = z.infer<typeof contactFormSchema>;
```

- Server Actions (`'use server'`) live in `actions/`, return plain UI-shaped objects, and convert failures into `{ ok: false, errorKey }` — never a thrown error crossing the wire.
- Route handlers that need cross-cutting checks use a **higher-order middleware wrapper**, not inline checks in every handler:

```ts
export const POST = withAuth(handleCreateOrder); // auth + validation before delegating
```

## Fetch hygiene

- **Every service fetch gets a timeout**: `signal: AbortSignal.timeout(10_000)` — a hung request becomes a caught `TimeoutError` that `toCaughtErrorKey` maps to `TIMEOUT` (vs `NETWORK` for a dead connection) instead of an infinite spinner. When a caller signal exists, combine: `AbortSignal.any([callerSignal, AbortSignal.timeout(ms)])`.
- **Writes that may fire around page exit use `keepalive: true`** — flushing a pending debounced save, "mark as read", analytics. The browser finishes the request after the tab closes. Constraints: the body is capped (~64 KiB) and the response is effectively fire-and-forget — never use it for reads.

## Separate decisions from actions

Pure functions decide (`calculateDiscount`, `buildSearchQuery`, `toSubscription`); thin shells act (call the API, set state, show the toast). The pure part is what gets unit tests — no mocking I/O to test logic.
