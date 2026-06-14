# active-maintenance-exceptions

Per `book/HARD-RULES.md` "Active-maintenance dep gate" — every direct dep must have a publish (any version, not just dist-tags) within 6 months. Exceptions documented here with rationale.

Audit tool: `sim/tools/dep-audit/audit.ts` reads `simdocs/deps.json`, hits `registry.npmjs.org`, computes `max(j.time[ver])` across every published version (excluding `created` / `modified` keys). The check answers "is the author still publishing anything?" — not "is the chosen tag fresh?".

Exception requires:
1. Verified rationale (stable problem / pure algorithm / no security surface)
2. OSS-import-first scan returned NO actively-maintained alternative (verified via dep-audit tool with `--names` flag)
3. Watch trigger for when the exception invalidates
4. Hand-roll alternative would be NET WORSE than maintaining the dep with stale-cadence

Never hand-roll when a maintained alternative exists, even lesser-known. Per memory `feedback_operator_patterns.md` "lesser-known + active > popular + dying".

## Exceptions

### remark-frontmatter

Latest: v5.0.0 (2023-09, 968d).

Reason: consumed transitively via `@next/mdx` for `/learn/*.mdx` frontmatter parsing. Part of the `unified` / `remark` / `rehype` ecosystem which IS actively maintained (operator pm4ai ALLOWED_STACK lists `remark`, `rehype`, `unified`). Frontmatter parsing is a stable problem; the spec doesn't change.

Alternative scan: `parse-frontmatter` v0.0.0 (56d) — too immature. `vfile-matter` (420d) — also stale. `gray-matter` banned per pm4ai.

Watch trigger: if `@next/mdx` swaps its frontmatter plugin upstream, follow.

### espresso-iisojs

Latest: v1.0.8 (2024-02, 810d).

Reason: pure Espresso-II Boolean minimization algorithm port (genieacs, MIT, native TypeScript). No security surface, deterministic + tested. Algorithm itself is canonical from Espresso-II reference paper. Actively wired in `packages/boolean/src/espresso.ts` for POS at width > 4 — handroll Q-M/Petrick blows up exponentially on dense maxterm sets (62 maxterms at width 6 hangs); espresso heuristic stays polynomial.

Alternative scan: zero actively-maintained JS QM/Espresso libraries exist (verified May 2026).

Watch trigger: any maintained fork ships; OR 12+-var workload profile pain forces WASM SIMD port (classabbyamp/espresso-logic via Emscripten).

### @react-three/offscreen

Latest: v1.0.0-rc.1 (2025-01, 469d).

Reason: canonical pmndrs solution for OffscreenCanvas + R3F. Handles DOM event forwarding, document/window shims, auto-fallback to main-thread render. No community fork exists. pmndrs cadence is slow but the package is the standard, not abandoned.

Alternative scan: no maintained alternative for R3F-in-Worker (verified May 2026).

Hand-roll would be NET WORSE: ~500 LOC to replicate event forwarding + shim layer; pmndrs maintains tests + edge cases.

Watch trigger: pmndrs ships v1.0.0 GA (rc.1 → 1.0.0); OR @react-three/fiber subsumes it.

### detect-gpu

Latest: v5.0.70 (2025-02, 445d).

Reason: GPU tier detection via benchmark DB lookup. The benchmark DB is the value; the code is thin. DB is ~12 months stale but library still works as scaffold.

Alternative scan: zero maintained alternatives. Hardware detection is a niche space.

Mitigation per `adr/adaptive-quality.md`: augment `detect-gpu` result with `hardwareConcurrency` + `deviceMemory` heuristic + user override. The library's output is one input to the tier score, not the sole decision.

Watch trigger: any maintained GPU-detection library ships; OR W3C `navigator.gpu.requestAdapter().info` standardizes GPU model identity in browser (currently spec-only).

### next-themes

Latest: v0.4.6 (2025-03, 428d).

Reason: thin wrapper around `matchMedia('(prefers-color-scheme: dark)')` + localStorage + html class manipulation. Operator pm4ai-stack ALLOWED. The dep doesn't accumulate bugs because the underlying primitives don't change.

Alternative scan: `use-system-theme` (1996d dead), `theme-ui` (131d active but heavy CSS-in-JS bundle banned by pm4ai), browser-native (would require ~50 LOC re-implementation).

Hand-roll would be NET WORSE: pm4ai stack alignment + tested edge cases (SSR hydration mismatch) > re-deriving locally.

Watch trigger: pm4ai upstream swaps it; OR Next.js ships native theme primitive.

### n8ao

Latest: v1.10.1 (2025-06, 329d).

Reason: SSAO postprocessing by N8python. Stable problem (screen-space AO algorithm). WebGPU path uses three's native TSL SSAO node (active in three.js); n8ao only used on WebGL2 fallback path per `adr/three-stack.md`.

Watch trigger: WebGL2 fallback retired (deprecates n8ao); OR maintained collective forks.

### simple-git-hooks

Latest: v2.13.1 (2025-07, 286d).

Reason: operator pm4ai-stack locked. Git hook wiring is a stable problem (write file at `.git/hooks/*` + run shell). Cadence is slow because nothing needs to change.

Watch trigger: pm4ai upstream swaps it.

## Verified active-OSS alternatives chosen instead of hand-roll

Per the "never reinvent the wheel" rule + verified via dep-audit `--names` scan May 2026:

| Concern | Stale candidate | Active OSS chosen |
|---|---|---|
| Topological sort | `toposort` v2.0.2 (2937d dead) | `@thi.ng/dgraph` v2.1.205 (3d) — thi.ng ecosystem |
| Canonical JSON serialization | `safe-stable-stringify` v2.5.0 (627d) | `canonicalize` v3.0.0 (35d) — RFC 8785 JCS-compliant |
| State diff | `microdiff` v1.5.0 (505d) | `rfc6902` v5.2.0 (76d) — JSON Patch RFC standard |
| Micro-benchmarks | `mitata` v1.0.34 (464d) | `tinybench` (today, active) |
| Color science | `culori` v4.0.2 (320d) | `colorjs.io` v0.6.1 (118d) — spec-author lib |
| Fuzzy search | `minisearch` v7.2.0 (239d) | `@orama/orama` v3.1.18 (145d) |
| Lighthouse CI | `@lhci/cli` v0.15.1 (322d) | `unlighthouse` v0.17.9 (28d) |
| Structured logging | `consola` v3.4.2 (421d) | `@logtape/logtape` v2.0.7 (9d) |
| Drag-and-drop legacy | `@dnd-kit/{core,sortable,utilities}` (524-919d) | `@dnd-kit/{react,abstract,dom,state,helpers}` v0.4.0 (30d) — same author, active line |

## Discipline

- Exceptions are NOT permanent. Quarterly audit re-verifies each rationale + alternative scan.
- New stale dep → run `bun tools/dep-audit/audit.ts --names a,b,c` against candidate alternatives BEFORE proposing hand-roll.
- Hand-roll only when: scan returns nothing maintained AND code is ≤30 LOC trivial threshold AND no security surface.
- Removing an exception (rationale invalidates / alternative ships) = one-line edit + dep swap.

## Caught by

- `sim/tools/dep-audit/audit.ts` on every push (pre-deploy gate)
- Quarterly refresh re-verifies each exception
- Operator pushback when an alternative the agent missed gets named — extends memory `feedback_operator_patterns.md` rules
