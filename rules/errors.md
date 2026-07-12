# Error Handling

**Errors are codes, not sentences.** Every failure surfaces to the frontend as a stable SCREAMING_SNAKE_CASE **error key**; the frontend decides what to show based on that key — which keeps copy out of services and stays i18n-ready.

## Error keys

- Keys are a **string-literal union** centralized in one module — the union is the source of truth and the value is the name; no const object, no second spelling to keep in sync. Namespace by prefix (`UPLOAD_*`); shared keys stay bare:

```ts
// constants/error-keys.ts
export type ErrorKey =
  | 'NETWORK'
  | 'UNAUTHORIZED'
  | 'UPLOAD_FILE_TOO_LARGE'
  | 'UPLOAD_FORMAT_NOT_SUPPORTED'
  | 'INVOICE_FETCH_FAILED';
```

- Keys are a **wire contract**, deliberately decoupled from any translation-file layout — reorganizing dictionaries never changes a key.
- Services, actions, and hooks return `{ ok: false, errorKey: 'INVOICE_FETCH_FAILED' }` — the literal is typo-safe because `Result<T>` types it. Never a human-readable message, never a raw `Error`:

```ts
// types/result.ts — the one shared definition, imported everywhere
type Result<T> = { ok: true; data: T } | { ok: false; errorKey: ErrorKey };
```

- The frontend resolves the key to copy at render time (i18n dictionary, toast, inline field error). i18n apps keep a flat `errors` section in the dictionary; single-language apps use an exhaustive record — TS then forces copy for every key:

```tsx
{isError ? <p role="alert">{t(`errors.${errorKey}`)}</p> : null}
```

```ts
// single-language apps — adding a key without copy is a compile error
export const errorCopy: Record<ErrorKey, string> = {
  NETWORK: 'Connection failed. Check your internet and try again.',
  UNAUTHORIZED: 'You need to sign in to do that.',
  UPLOAD_FILE_TOO_LARGE: 'That file is too large.',
  UPLOAD_FORMAT_NOT_SUPPORTED: 'That file format is not supported.',
  INVOICE_FETCH_FAILED: 'Could not load the invoice.',
};
```

- Raw error details (stack, vendor payload) are **logged where they happen** and go no further.

## Guard clauses first, happy path unindented

Validate invariants at the top and throw/return early; avoid `else`:

```ts
if (!portalUrl) throw new Error('portalUrl is required');
if (!response.ok) return { ok: false, errorKey: 'INVOICE_FETCH_FAILED' };
```

Two failure kinds, two channels: **invariant violations** (programmer error — throw with a dev-facing message, like the `portalUrl` guard; it never reaches the UI) vs **expected failures** (user-visible — return `{ ok: false, errorKey }`). Keys are only for the second kind.

## try/catch at every I/O boundary

Log with the `'Error in <fn>::'` prefix so failures are greppable, then return the key — errors never propagate raw into the UI:

```ts
export const fetchInvoice = async (id: string): Promise<Result<Invoice>> => {
  try {
    const response = await fetch(`${env.API_BASE_URL}/invoices/${id}`, { signal: AbortSignal.timeout(10_000) });
    if (!response.ok) return { ok: false, errorKey: 'INVOICE_FETCH_FAILED' }; // server said no — operation-specific key
    return { ok: true, data: toInvoice(await response.json()) };
  } catch (error) {
    console.error('Error in fetchInvoice::', error);
    return { ok: false, errorKey: 'NETWORK' }; // request never completed — network
  }
};
```

## Narrow before reading

`error` is `unknown`; check `instanceof Error` when the log needs details. The UI still only ever receives a key:

```ts
} catch (error) {
  console.error('Error in submitOrder::', error instanceof Error ? error.message : error);
  setState({ status: 'error', errorKey: 'NETWORK' });
}
```

## Surfacing to the user

Failures reach the user through the state machine (`{ status: 'error', errorKey }`) or a toast fed by the resolved copy (``t(`errors.${errorKey}`)`` / `errorCopy[errorKey]`) — never an unhandled rejection, never an inline hardcoded string.
