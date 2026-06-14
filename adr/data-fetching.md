# data-fetching

## Decision

Pure-client app: no backend, no server-side persistence. Data is either static (built at compile time), computed on demand (pure functions), or read from the visitor's own browser.

| Need | Primitive |
|---|---|
| Initial page render | RSC |
| Heavy compute (e.g. K-map QM ≥6 var) | Server Action with `'use cache'`, keyed on input hash |
| Client compute memoization | `@tanstack/react-query` keyed on input hash |
| Local snapshot list (`/me`) | Client read of `localStorage` |
| Shared snapshot load (`/s/[hash]`) | Client decode of URL fragment |
| OG card | Route Handler with `ImageResponse` |
| RUM telemetry | Route Handler `/api/rum` fire-and-forget (cookieless, aggregate) |
| Health check | Route Handler `/api/healthz` per `adr/health-check.md` |
| Search index | Build-time JSON + client lazy |

Per `DATA-FETCHING.md` decision matrix.

## Why Server Actions for compute

- Progressive enhancement (forms work without JS)
- Typed validation at boundary via Zod
- Co-located with components that submit them
- Error envelope per `API-CONVENTIONS.md`
- React 19 patterns (`useActionState`, `useOptimistic`) integrate naturally
- Server Actions hold the heavy pure assemblers/solvers off the client bundle, cached by input hash

## Why RSC over client fetch for initial data

- Smallest client bundle
- SSR streaming (Suspense)
- SEO-friendly per `adr/seo-metadata.md`
- Server-only deps (heavy assemblers, content readers) stay server-side

## Local + shared state

- Snapshots live in the visitor's browser (`localStorage`); the `/me` list is a client read
- Shares carry the whole state in the URL fragment; `/s/[hash]` decodes it client-side, no network round-trip after the static app loads
- States too large for a URL fragment are tier `'oversize'` — non-shareable by design, no server fallback

## Cache strategy

Per `DATA-FETCHING.md`. Content-addressed compute paths get forever-cache; health endpoints get `no-store`; static routes get standard ISR-equivalent.

## Caught by

- `tools/lint/no-client-fetch.ts` greps client components for raw `fetch()` outside Server Action consumers + the OG/RUM/healthz handlers
- Smoke per surface: expected cache headers present
- E2E: optimistic updates roll back on Server Action error
