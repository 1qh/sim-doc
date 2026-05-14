# active-maintenance-exceptions

Per `book/HARD-RULES.md` "Active-maintenance dep gate" — every direct dep must have a publish (any dist-tag) within 6 months. This ADR lists the documented exceptions + rationale.

Audit tool: `sim/tools/dep-audit/audit.ts` reads `simdocs/deps.json`, hits `registry.npmjs.org` for each, computes `max(time[ver])` across ALL `dist-tags` (latest / next / beta / alpha / canary / rc / etc.). A package counts as active if ANY tag publish is within 6 months.

## Exceptions (verified 2026-05-14)

### espresso-iisojs

Latest: v1.0.8 (2024-02, 810d).

Reason: pure Espresso-II Boolean minimization algorithm port (genieacs, MIT, native TypeScript). No security surface, deterministic + tested, no maintained JS alternative. Algorithm itself is canonical from Espresso-II reference paper — no logic churn expected.

Watch trigger: any maintained fork ships; OR 12+-var workload profile pain forces WASM SIMD port (classabbyamp/espresso-logic via Emscripten).

### n8ao

Latest: v1.10.1 (2025-06, 329d).

Reason: SSAO postprocessing implementation by N8python. Stable problem (screen-space AO algorithm); single-author cadence is acceptable for an effect implementation. WebGPU path uses three's native TSL SSAO node (active alternative within three.js); n8ao only used on WebGL2 fallback path per `adr/three-stack.md`.

Watch trigger: WebGL2 fallback path retired (would deprecate n8ao); OR fork by maintained collective.

### simple-git-hooks

Latest: v2.13.1 (2025-07, 286d).

Reason: operator pm4ai-stack locked. Git hook wiring is a stable problem (file at `.git/hooks/*`, run shell). Cadence is slow because nothing needs to change.

Watch trigger: pm4ai upstream swaps it.

### @auth/core

Latest: v0.34.3 (2025-10, 196d).

Reason: consumed transitively via `@convex-dev/auth` (active, byerag operator pattern). The `@auth/*` ecosystem is broadly active — `@auth/express` v0.12.2 + `@auth/sveltekit` v1.11.2 both 29d. `@auth/core` itself is on slower cadence because it's the underlying engine other auth wrappers depend on.

Watch trigger: better-auth (pm4ai's preferred default) ships a Convex adapter with feature parity to `@convex-dev/auth`. At that point migrate the byerag pattern + this project together.

## Discipline

- Exceptions are NOT permanent — every quarterly audit re-verifies the rationale still holds
- New stale dep → first scan ecosystem (`@scope/*` siblings + `next`/`beta`/`alpha`/`canary` dist-tags) BEFORE proposing replacement, per memory rule
- Exception requires: (a) rationale, (b) verified no maintained alternative or active fork, (c) watch trigger
- Removing an exception (because rationale invalidates) is a one-line edit to this ADR + dep swap commit

## Caught by

- `sim/tools/dep-audit/audit.ts` runs on every push (pre-deploy gate)
- Quarterly refresh: re-verify each exception's rationale + last-publish date
- Operator pushback when an alternative the agent missed gets named — extends memory `feedback_operator_patterns.md` "ecosystem migration check" rule
