---
name: px-form
description: Build a form the house way — zod schema first, react-hook-form logic, shadcn Field/FieldGroup markup, control chooser, submit via Result-returning action/service. Use when creating or extending forms, validation, or form submit flows.
---

# Form

End-to-end form recipe. Markup and control rules in [references/forms.md](references/forms.md). Load `px-conventions` for errors, components, and hooks-state when wiring submit.

## 1. Decide — form library or not

| Situation | Approach |
| --- | --- |
| 2+ fields, validation rules, or submit state | **RHF + zod** — follow this skill |
| Single uncontrolled input (search, inline rename) | Plain state or a form action — no form library |

Inspect the repo: reuse an existing form pattern, schema location, and submit hook before inventing a parallel one.

## 2. Schema first — before markup

- Export a **zod schema** colocated with the feature (`lib/` or next to the form component).
- Derive the form type: `type CheckoutFormValues = z.infer<typeof checkoutSchema>`.
- List **validation failures** (field-level, from zod) and **submit failures** (server/network) separately.
- Submit failures use SCREAMING_SNAKE `ErrorKey` literals — add them to the feature's `constants/error-keys.ts` before wiring submit (`px-conventions`: `errors` rule).

**Present the schema, field list, and error keys** — confirm before building UI.

## 3. Choose controls

Pick from the control chooser in [references/forms.md](references/forms.md). Reuse shadcn/ui primitives from the project's UI layer — never raw `div` + `space-y-*` form markup.

| Need | Control |
| --- | --- |
| Simple text | `Input` |
| Dropdown | `Select` |
| Searchable dropdown | `Combobox` |
| Boolean in settings | `Switch` |
| Boolean in a form | `Checkbox` |
| Single choice, few options | `RadioGroup` |
| Toggle 2–5 options | `ToggleGroup` |
| Multi-line | `Textarea` |
| OTP | `InputOTP` |

Buttons inside inputs: `InputGroup` + `InputGroupAddon`. Related checkboxes/radios: `FieldSet` + `FieldLegend`.

## 4. Markup — FieldGroup / Field

- `'use client'` on the form component leaf only.
- Wrap fields in `FieldGroup` → `Field` → `FieldLabel` + control + optional `FieldDescription`.
- Invalid/disabled: **`data-invalid` / `data-disabled` on `Field`** and **`aria-invalid` / `disabled` on the control**.
- Connect RHF: `register` or `Controller`/`FormField` — match what the repo already uses.
- Submit errors: render from `errorKey` at display time — never hardcoded strings.

```tsx
<FieldGroup>
  <Field data-invalid={!!errors.email}>
    <FieldLabel htmlFor="email">Email</FieldLabel>
    <Input id="email" type="email" aria-invalid={!!errors.email} {...register('email')} />
    {errors.email ? <FieldDescription role="alert">{errors.email.message}</FieldDescription> : null}
  </Field>
</FieldGroup>
```

## 5. Logic — useForm + submit

```ts
const form = useForm<CheckoutFormValues>({
  defaultValues,
  mode: 'onChange',
  resolver: zodResolver(checkoutSchema),
});
```

- Guard double-submit with a per-button async hook (`{ isProcessing, execute }`) or `form.formState.isSubmitting` — not disabled-flag spaghetti.
- Submit handler calls a **service or server action** returning `Result<T, K>` — never catch-and-toast a thrown error from the boundary.
- On `{ ok: false, errorKey }`: set form/root error state or toast from resolved copy; on `{ ok: true }`: reset or redirect per the journey.

## 6. File layout

Colocate with the feature (route or `features/<name>/`):

```
components/checkout-form.tsx    # 'use client' — markup + useForm
lib/checkout-schema.ts          # zod schema + z.infer type (or schema next to form if tiny)
actions/submit-checkout.ts      # 'use server' — thin shell → service
services/checkout.ts            # external call, DTO map, Result return
constants/error-keys.ts         # CheckoutErrorKey union
```

Kebab-case filenames, named arrow-const export, no barrel `index.ts`.

## 7. Verify

- Every field validates per schema; invalid states show on the correct `Field`.
- Submit: loading/disabled during `isProcessing`; success and each `errorKey` path from phase 2.
- No hardcoded user-facing error strings; typecheck passes with narrowed `Result<T, K>`.
- Final test: would a senior engineer call this overcomplicated?
