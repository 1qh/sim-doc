# state-management

## Decision

Layered state with strict per-layer ownership:

| Layer | Owns |
|---|---|
| Convex backend | Cross-device persisted state |
| URL state | Shareable + bookmarkable state |
| localStorage | Per-device preferences |
| IndexedDB | Offline queue + cached bundles |
| Zustand stores | Hot-loop sim state + ephemeral UI |
| React state | Local component state |
| sim-engine | Pure-function transformations (stateless) |

Per `STATE-MANAGEMENT.md`.

## Why zustand for hot-loop state

- Subscribe outside React render via `subscribeWithSelector` — frame-loop mutations don't thrash React
- Tiny bundle (~1 KB)
- No provider boilerplate
- Selector pattern matches the access discipline needed for `useFrame` loops
- Active maintenance + operator-stack precedent

## Rejected alternatives

- **Redux / Redux Toolkit** — heavier ceremony, no benefit over zustand for this scope
- **Jotai / Recoil** — atom-shaped state pattern doesn't fit sim-engine's whole-state shape
- **MobX** — proxy-based reactivity fights with R3F transient subscribes
- **Pure React (Context + useReducer)** — re-render thrash in hot loops
- **Convex for all state** — latency cost for ephemeral UI is unacceptable
- **TanStack Query / SWR for server state** — Convex client already provides reactive cache + subscriptions

## Why URL state for sharing

- URL is the natural permalink surface — content-addressed snapshots live there per `adr/share-content-addressed.md`
- Browser back/forward works correctly
- Linkable / bookmarkable by definition
- SSR-friendly (server reads `searchParams` directly)

## Anti-patterns banned

- Persisting frame-loop state to localStorage (storage thrash)
- Storing URL-derivable state in zustand (duplication, drift)
- React state for cross-component sim state (lift to zustand)
- Convex mutation for ephemeral UI (latency, cost, wrong layer)
- Server state mirrored into Zustand (Convex client owns server state)

## Caught by

- `tools/lint/state-layer.ts` enforces per-layer ownership patterns
- E2E: refresh-survives-URL-state, refresh-resets-zustand
- Smoke: localStorage writes blocked inside hot loop
