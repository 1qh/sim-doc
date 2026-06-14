# FORM-HANDLING

React 19 Actions + `useActionState` + `useOptimistic` + `useFormStatus`. Progressive enhancement by default — every form works without JavaScript.

## Pattern

```tsx
// app/kmap/actions.ts
'use server';

import { z } from 'zod';

const SaveKmapSchema = z.object({
  vars: z.number().int().min(2).max(6),
  minterms: z.array(z.number().int().min(0)),
  groupings: z.array(z.array(z.number())),
});

export async function saveKmapAction(_prev: ActionState, formData: FormData) {
  const parsed = SaveKmapSchema.safeParse({
    vars: Number(formData.get('vars')),
    minterms: JSON.parse(formData.get('minterms') as string),
    groupings: JSON.parse(formData.get('groupings') as string),
  });
  if (!parsed.success) return { ok: false, errCode: 'BAD_INPUT', fieldErrors: parsed.error.flatten() };
  // ... canonicalize + encode to URL fragment
  return { ok: true, fragment: '...' };
}
```

```tsx
// app/kmap/SaveButton.tsx
'use client';
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';
import { saveKmapAction } from './actions';

export function SaveButton({ kmap }) {
  const [state, action] = useActionState(saveKmapAction, null);
  return (
    <form action={action}>
      <input type="hidden" name="vars" value={kmap.vars} />
      <input type="hidden" name="minterms" value={JSON.stringify(kmap.minterms)} />
      <input type="hidden" name="groupings" value={JSON.stringify(kmap.groupings)} />
      <Submit />
      {state?.ok === false && <ErrorBanner code={state.errCode} fieldErrors={state.fieldErrors} />}
      {state?.ok === true && <ShareLink fragment={state.fragment} />}
    </form>
  );
}

function Submit() {
  const { pending } = useFormStatus();
  return <Button type="submit" disabled={pending}>Save</Button>;
}
```

## Patterns enforced

### Server Action signature

```ts
type ActionState<TData = unknown> =
  | null
  | { ok: true; data: TData }
  | { ok: false; errCode: ErrorCode; fieldErrors?: Record<string, string[]>; message?: string };

type ServerAction<TData = unknown> = (prev: ActionState<TData>, formData: FormData) => Promise<ActionState<TData>>;
```

Typed end-to-end. Per `ERROR-CATALOG.md` error codes.

### Validation

- Zod at the boundary per `book/HARD-RULES.md` "Maximum typesafety"
- Server-side validation is the SSOT; client may also validate for instant feedback but server is final
- `safeParse` → typed error response; never throws to user
- Field errors mapped back to specific input names via `fieldErrors`

### Optimistic updates

```tsx
const [snapshots, optimisticAdd] = useOptimistic(serverSnapshots, (curr, draft) => [...curr, draft]);

const action = async (formData) => {
  optimisticAdd({ hash: 'pending', createdAt: Date.now() });
  const result = await saveKmapAction(null, formData);
  // server result arrives via useActionState
};
```

Pattern: optimistic state appears instantly; server confirms or rolls back via revalidation.

### Pending UI

`useFormStatus` inside a form gives `{ pending, data, method, action }` for the parent form. Components like `<Submit>` consume to disable / show spinner without prop-drilling.

### Field-level error display

```tsx
const fieldError = state?.fieldErrors?.[fieldName]?.[0];
<TextField error={fieldError} aria-invalid={!!fieldError} aria-describedby={fieldError ? `${fieldName}-err` : undefined} />
{fieldError && <span id={`${fieldName}-err`} role="alert">{fieldError}</span>}
```

Per `A11Y.md` ARIA practices.

### Progressive enhancement

Every form's `action` attribute points to a Server Action. Without JavaScript:
- Form submits as standard POST
- Server Action processes
- Response streams back as new page render with `state` interpreted from form-data echo
- All forms remain functional

With JavaScript:
- `useActionState` intercepts submission
- No page reload
- Optimistic updates feel instant
- Pending state via `useFormStatus`

## Anti-patterns banned

- Client-only forms that don't degrade gracefully
- `onSubmit` handlers that bypass Server Actions
- Ad-hoc client fetch for compute when a Server Action exists (use the Server Action — gives progressive enhancement, error envelope, typed state)
- Hidden validation gaps (always validate server-side, never trust client-only)
- `e.preventDefault()` patterns (use `useActionState`)
- Bare `<button type="submit">` without `<Submit>` wrapper that handles pending state

## Caught by

- `tools/lint/forms-progressive.ts` greps for `<form>` elements; asserts `action` prop points to a server action or RSC-mounted endpoint
- E2E with JS disabled: every form submits + processes + redirects correctly
- Validation smoke: invalid input returns typed `BadInputError` with field errors
- A11y test: form errors announced via aria-live regions
