# Hooks & State

## Custom hooks

- **Hooks return an object, never a tuple** — expose `status` itself (consumers compare or `switch` exhaustively) plus only the derived flags call-sites actually read. Booleans are always derived from `status`, never stored:

```ts
return {
  submit,
  status: state.status,
  isLoading: state.status === 'loading',
  errorKey: state.status === 'error' ? state.errorKey : undefined,
};
```

- **Async/UI state is one `status` union in one `useState`** — a bare union by default, a discriminated union when data/`errorKey` must be tied to the state (see [typescript.md](typescript.md)); never parallel `isLoading`/`isError`/`isSuccess` booleans that can contradict. Failures carry an **error key**, not a message string (see [errors.md](errors.md)).
- **Never mirror server state into `useState` without reconciliation** (`useState(propFromServer)`) — the prop is read once and the copy goes stale on the next refetch. Optimistic UI runs on `useOptimistic` + `useTransition` — a derived overlay over the last server-confirmed value; the API response decides what stays (see [optimistic-ui.md](optimistic-ui.md)).
- Side-effect hooks always clean up on unmount.

## Context

- Context via a `createSafeContext` factory that throws when the provider is missing — no silently-undefined contexts:

```tsx
export function createSafeContext<ContextValue>(errorMessage: string) {
  const Context = createContext<ContextValue | null>(null);
  const useSafeContext = () => {
    const ctx = useContext(Context);
    if (ctx === null) throw new Error(errorMessage);
    return ctx;
  };
  const Provider = ({ children, value }: { children: React.ReactNode; value: ContextValue }) => (
    <Context.Provider value={value}>{children}</Context.Provider>
  );
  return [Provider, useSafeContext] as const;
}

const [CheckoutProvider, useCheckout] = createSafeContext<CheckoutContextValue>(
  'useCheckout must be used within a CheckoutProvider'
);
```

**When to use context:** state genuinely shared by a subtree (auth/session, theme, an active checkout) that many descendants read. **When not:** to skip one level of props — compose via `children` first (see [components.md](components.md)); and never for server data a Server Component could fetch where it's used.
