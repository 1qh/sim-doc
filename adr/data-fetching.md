# data-fetching

## Decision

| Need | Primitive |
|---|---|
| Initial page render | RSC |
| Real-time subscriptions | Convex reactive query (client) |
| Mutations | Server Action (writes go through here, not direct Convex client) |
| Heavy compute (e.g. K-map QM ≥6 var) | Server Action with `'use cache'` |
| Snapshot load | RSC + CF edge cache |
| OG card | Route Handler with `ImageResponse` |
| RUM telemetry | Route Handler `/api/rum` fire-and-forget |
| Health check | Route Handler `/api/healthz` per `adr/health-check.md` |
| Search index | Build-time JSON + client lazy |

Per `DATA-FETCHING.md` decision matrix.

## Why Server Actions over direct Convex client mutations

- Progressive enhancement (forms work without JS)
- Typed validation at boundary via Zod
- Co-located with components that submit them
- Error envelope per `API-CONVENTIONS.md`
- React 19 patterns (`useActionState`, `useOptimistic`) integrate naturally
- Server Actions can compose multiple Convex operations atomically

Direct Convex client mutations are reserved for:
- Reactive subscriptions (Convex client owns those by design)
- Already-validated trusted server-side calls inside other Server Actions

## Why RSC over client fetch for initial data

- Smallest client bundle
- SSR streaming (Suspense)
- SEO-friendly per `adr/seo-metadata.md`
- Server-only deps (heavy assemblers, content readers) stay server-side
- Authentication state read server-side from cookie

## Rejected alternatives

- **TanStack Query / SWR** — Convex client already provides reactive cache; adding TanStack duplicates that surface
- **GraphQL** — Convex's typed function interface is the GraphQL-shaped equivalent without runtime cost
- **REST routes for all data** — losing typed end-to-end safety + reactive subscriptions

## Cache strategy

Per `DATA-FETCHING.md`. Content-addressed paths get forever-cache; auth-sensitive paths get `no-store`; static routes get standard ISR-equivalent.

## Caught by

- `tools/lint/no-client-fetch.ts` greps client components for raw `fetch()` outside Convex / Server Action consumers
- Smoke per surface: expected cache headers present
- E2E: optimistic updates roll back on server error
