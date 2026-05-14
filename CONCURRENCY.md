# CONCURRENCY

Maximum concurrency + parallelism contract. Every compute path that can run in parallel does. Every wait that can overlap with other work does.

## Worker pool

| Worker | Count | Spawn |
|---|---|---|
| Render worker (OffscreenCanvas) | 1 per active 3D scene (compare mode = 2) | App boot |
| Solver worker (QM / Petrick / Espresso) | `min(navigator.hardwareConcurrency - 1, 4)` with floor 2 | App boot (pre-warm) |
| Pipeline analyzer worker | 1 | App boot (pre-warm) |
| Assembler worker | 1 | First parse beyond inline threshold |
| Shared worker (cross-tab compute reuse) | 1 per origin | First tab opens it |

All workers pre-warmed at app boot per `PERFORMANCE.md` "Pre-warmed Worker pool" rule.

## Parallel solver via truth-table partitioning

QM (and Espresso) for ≥ 5-variable functions shard the truth-table across N solver workers:

```mermaid
flowchart LR
    Input[(Truth table 2^N)] --> Split[Partition into K shards]
    Split --> W1[Worker 1 — find PIs in shard 1]
    Split --> W2[Worker 2 — find PIs in shard 2]
    Split --> WK[Worker K — find PIs in shard K]
    W1 --> Merge[Main / coordinator — merge PI list, dedup, Petrick selection]
    W2 --> Merge
    WK --> Merge
    Merge --> Result[(Minimal cover)]
```

Coordinator runs Petrick selection (the merge step) on the main solver worker — partition is the parallelizable phase, selection is dependency-heavy and stays serial. Net speedup ≈ K for the partition phase, total speedup ≈ 0.7 × K accounting for serial selection.

## Per-scene render worker in compare mode

Compare mode mounts 2 scenes side-by-side. Each gets its own `OffscreenCanvas` + dedicated render worker. Both workers receive sync step messages but render independently.

```mermaid
flowchart LR
    MainThread[Main thread — input + UI + zustand] -->|sync step| LeftWorker[Render worker left]
    MainThread -->|sync step| RightWorker[Render worker right]
    LeftWorker --> LeftCanvas[Left OffscreenCanvas]
    RightWorker --> RightCanvas[Right OffscreenCanvas]
```

CPU cost is ~2× single scene, but parallel — wall-clock matches single scene on multi-core devices.

## Parallel Convex query discipline

Independent queries always parallel:

```ts
// good
const [snapshots, profile, examples] = await Promise.all([
  convex.query(api.snapshots.mySnapshots),
  convex.query(api.users.profile),
  convex.query(api.examples.list),
]);

// banned — sequential await for independent reads
const snapshots = await convex.query(api.snapshots.mySnapshots);  // 🚫
const profile = await convex.query(api.users.profile);
```

Lint asserts: multiple `await convex.query(...)` in the same function body without intermediate dependency = violation; replace with `Promise.all`.

## Concurrent Server Actions

Same pattern for client-initiated Server Action chains where actions don't depend on each other:

```ts
const [savedKmap, fetchedExamples] = await Promise.all([
  saveKmapAction(state),
  loadExamplesAction(),
]);
```

## Selective hydration priority

React 19 supports priority-shaped hydration. Visible content hydrates first:

```tsx
<Suspense fallback={<SkeletonHero />}>
  <HeroAboveFold />  {/* hydrates first */}
</Suspense>
<Suspense fallback={null}>
  <SidebarBelowFold />  {/* hydrates after visible-priority items */}
</Suspense>
```

Hydration priority signaled via Suspense boundary placement + `<link rel="modulepreload" fetchpriority="high">` on critical bundles.

## Streaming hash for large snapshots

`blake3` supports incremental hashing — chunks fed as stream arrives. For snapshot bodies > 64 KB:

```ts
const hasher = blake3.create();
for await (const chunk of stream) {
  hasher.update(chunk);
  // chunk also feeds incremental zstd decompress in parallel
}
const hash = hasher.digest();
```

Hash compute overlaps with stream arrival; net wall-clock = stream latency only.

## Cross-tab coordination

| Mechanism | Use |
|---|---|
| `BroadcastChannel('sim')` | Signin event propagates to all tabs; save event propagates so other tabs invalidate their localStorage anon-hash list |
| `SharedWorker` (where supported) | Single QM solver result shared across tabs (cache hit even if tab A solved, tab B requests same hash) |
| `navigator.locks.request('save', ...)` | Web Locks API — prevent two tabs from simultaneously saving identical state (content-addressed dedups anyway, but lock saves the redundant write) |
| `localStorage` event | Fallback for browsers without BroadcastChannel (mostly redundant; covered by BroadcastChannel polyfill) |

## GPU / CPU pipelining

CPU prepares next-frame transform matrices + animation interpolation while GPU renders current frame:

```ts
useFrame((state) => {
  // CPU work: update next-frame state
  prepareNextFrame(state);
  // GPU autoflows — render call submitted, returns immediately
  // Browser pipelines: while GPU draws, CPU is free for next frame's prep
});
```

drei `PerformanceMonitor` ensures GPU saturation doesn't tank CPU-side budget.

## Build graph parallelism

Turbo's task graph determines parallel execution. Discipline: every package's `dependsOn` array names only TRUE dependencies. Spurious dependencies serialize parallel work.

```jsonc
// good — package A only waits on real deps
{ "pipeline": { "build": { "dependsOn": ["^build"] } } }

// banned — overly broad dependsOn that forces unnecessary serialization
```

Turbo's `tasks` should show maximum parallelism for `bun run build` on a multi-core machine. CI verifies build graph fans out wide.

## Test parallelism

- Bun test: parallel by default, per-file isolation
- Per-package tests run in parallel via Turbo
- E2E (Playwright): parallel workers, shared browser instance with isolated contexts
- Convex integration tests: per-test isolated Convex in-memory backend

## CI parallelism (post velocity-mode exit)

GitHub Actions matrix:
- Lint + typecheck: one job per workspace, parallel
- Unit tests: one job per package, parallel
- E2E tests: sharded across N runners
- Lighthouse-CI: per-route parallel
- Mutation tests: per-package parallel
- Visual regression: per-fixture parallel

Total wall-clock = slowest single job, not sum.

## Speculative pre-compute on idle

`requestIdleCallback` fires for non-critical compute. Patterns:
- Pre-compute next N frames during scrub direction (per `PERFORMANCE.md` predictive input)
- Pre-warm next-likely route's data via Convex queries while user reads current
- Pre-index search corpus updates after content change
- Pre-validate next likely user input (e.g., next instruction in a curriculum sequence)

Idle work cancellable via `AbortSignal` if user navigation / interaction supersedes.

## Concurrent telemetry flush

`/api/rum` accepts batched events; client buffers + flushes on `requestIdleCallback` or `visibilitychange` (whichever fires first). Never blocks input or render.

`navigator.sendBeacon` for unload-time flush so events ship even on tab close.

## Concurrent file I/O on server

Server Actions + Route Handlers always `Promise.all` for independent fs / Convex / network ops.

## Anti-patterns banned

- Sequential `await` on independent operations (use `Promise.all`)
- Sync work blocking hydration (always Suspense-bounded)
- Single worker handling work that could shard
- Forgetting to pass `AbortSignal` to async ops in idle callbacks
- Cross-tab state drift (BroadcastChannel + localStorage event)
- `setTimeout` for delay between work units (use `scheduler.yield()` + priority)
- Saving without `navigator.locks.request` when concurrent tabs likely
- Spurious `dependsOn` in turbo config serializing parallel work

## Caught by

- `tools/lint/promise-all-discipline.ts` — sequential `await` on independent operations flagged
- `tools/lint/worker-pool-sizing.ts` — workers spawned with correct count formula
- `tools/lint/abort-signal-coverage.ts` — every async exported fn declares `signal?: AbortSignal`
- Turbo `--graph` output verified maximum parallelism
- CI total wall-clock budget per stage
- BroadcastChannel smoke — open 2 tabs, save in one, verify other reflects
- SharedWorker smoke — QM result cache hit cross-tab
- Compare mode smoke — 2 render workers spawned, both meet frame budget
- Parallel solver smoke — 6-var QM completes within budget on K-worker partition
