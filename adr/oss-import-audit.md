# oss-import-audit

OSS-import-first scan results per non-trivial concern. Each entry: chosen OSS dep + rejection rationale for alternatives + active-maintenance evidence + license compatibility + bundle-size impact.

Per `book/PHILOSOPHY.md` "OSS-import first, hand-roll only when locked" + `book/HARD-RULES.md` "Active-maintenance dep gate" (every direct dep ships upstream release within 6 months).

## Scan policy

Hand-roll allowed only when:
- Code is ≤ 30 lines AND has no maintained OSS equivalent AND is genuinely trivial
- An OSS scan returned no library meeting active-maintenance gate
- An OSS lib exists but surface mismatch requires more adapter code than the underlying lib (rare)

Hand-roll bans:
- Cryptographic / hash code (`@noble/hashes` for blake3 ✓)
- Compression (`fzstd` / native `CompressionStream` ✓)
- Validation (`zod` v4 ✓)
- JSON canonicalization (use `safe-stable-stringify` or `canonicalize`, never hand-roll)
- Parser combinators (use `chevrotain` / `peggy` / `parsimmon`)
- Worker RPC (`Comlink` ✓)
- Worker pool (`workerpool` or equivalent)
- Fuzzy search (`mini-search` / `fuse.js` / `fzf-for-js`)
- Diff (`microdiff` / `deep-object-diff`)
- Toposort (`toposort`)
- Color manipulation (`culori` / `colord`)
- GPU detection (`detect-gpu`)
- Syntax highlighting (`shiki`)

## Per-concern audit

### Boolean minimization (QM / Petrick / Espresso / hazard analysis)

Scan candidates:
- `qm-solver` — npm, last release evaluated at scan time per active-maintenance gate
- `quine-mccluskey-js` — npm
- `logicminimizer.js` — npm
- `boolean-algebra-js` — npm
- `bool-minimize` — npm
- `karnaugh-map` — npm

Locked pick: TBD at scan time — agent runs npm search + checks each against active-maintenance gate + surface fit + license + TS types. Chosen lib wrapped by `packages/boolean` which adds:
- Typed input/output via Zod
- Cancellation via AbortSignal
- Worker-pool integration for partition strategy
- Cross-link helper for datapath truth-table ingestion

If NO library passes active-maintenance gate + surface fit: hand-roll inside `packages/boolean` is justified (the substrate becomes the canonical OSS contribution for this gap). ADR amended with rejection rationale per candidate.

### MIPS assembler / disassembler

Scan candidates:
- `mips-assembler` — npm
- `mipsjs` — npm
- `js-mips` — npm
- `mips-cpu` — npm

Locked pick: TBD at scan time. If chosen lib covers our locked instruction set (`MIPS-ISA.md` 17 encodable + ratchet to full MIPS32), import + wrap with our error-catalog typed errors + Monaco diagnostics adapter.

If NO library covers our locked ISA shape + ratchet path: hand-roll the assembler/encoder/decoder is justified (~300 LOC). The ISA is the contract per `MIPS-ISA.md`; codegen from spec.

### MIPS executor

Scan candidates:
- `node-mips` — npm
- `js-mips-simulator` — npm

Locked pick: typically hand-roll because our executor must produce per-step datapath traces (which paths/components activate per cycle), not just final state. Existing libraries return final register/memory state only. Adapter cost > write-from-scratch.

Hand-roll justified per scan rationale. Documented as the canonical executor for the locked datapath; codegen-driven from `MIPS-DATAPATH.md` per `adr/codegen-pipeline.md`.

### Snapshot canonicalize

Locked pick: `safe-stable-stringify` (most-imported, active, MIT). Wraps JSON canonical serialization with deterministic key order, no `undefined` handling surprises, BigInt support.

Fallback option: `canonicalize` (RFC 8785 JCS-compliant) if cross-platform interop with non-JS clients becomes a requirement.

Hand-roll banned per scan rationale (canonicalization correctness is too easy to break subtly).

### State diff (Sim worker → Render worker)

Locked pick: `microdiff` (smallest, fastest, active, MIT). Emits typed diff ops for object trees. Render worker applies diff to its local scene state.

Rejected alternatives:
- `deep-object-diff` — larger bundle, slower, redundant API
- `fast-diff` — string-diff only, wrong shape
- Hand-roll — bug-risk for nested typed arrays

### Boolean expression parser (K-map input)

Locked pick: `chevrotain` for typed AST + Monaco grammar integration. Active, fast, MIT.

Rejected alternatives:
- `peggy` — PEG generator, runtime parser generation, larger bundle
- `parsimmon` — combinator-based, less Monaco-friendly
- Hand-roll recursive-descent — would re-derive Pratt + error recovery + Monaco diagnostics; OSS-import-first scan favors chevrotain

### Fuzzy search (command palette)

Locked pick: `mini-search` (smallest, in-memory, fast, MIT, active). Single-file ≤ 30 KB. Suits ~500-item search index.

Rejected alternatives:
- `fuse.js` — heavier bundle, more features than needed
- `fzf-for-js` — port-quality varies, less maintained
- Native `String.includes` filter — works for trivial case but fuzz tolerance is the point

### Worker pool + work-stealing

Locked pick: `workerpool` (mature, MIT, active, ~10k weekly downloads, TS types) for base pool primitive.

Work-stealing extension layered on top — `workerpool` doesn't ship work-stealing natively; cross-pool steal is a thin adapter (`~50 LOC`) reading from sibling queues. Adapter justified since the pattern is too project-specific for generic OSS to fit cleanly.

Rejected alternatives:
- `threads.js` — heavier, more abstractions
- Hand-roll pool from scratch — re-derives lifecycle / message routing / queue logic that `workerpool` already battle-tests

### Device tier detection

Locked pick: `detect-gpu` (npm, MIT, active). Returns GPU tier 0-3 based on benchmark database keyed by GPU model string. Combined with `hardwareConcurrency` + `deviceMemory` for our `high`/`mid`/`low` tier per `adr/adaptive-quality.md`.

### Color manipulation (color-blind palette variants)

Locked pick: `culori` (most-comprehensive color science, supports OKLCH, MIT, active). Used to derive deuteranopia/protanopia/tritanopia accent shifts from base palette deterministically.

Rejected alternatives:
- `colord` — smaller bundle but less complete color-space coverage
- `chroma-js` — older, larger
- Hand-roll — color-space math is exactly the kind of thing OSS-import-first guards against

### Toposort (substrate dependency graph for build / foundation order checks)

Locked pick: `toposort` (npm, MIT, ~ten years active, trivial API). Used by `tools/lint/foundation-order.ts` to verify foundation-bootstrap-order.md phase graph is acyclic + reachable.

### MDX code-block syntax highlighting

Locked pick: `shiki` (microsoft, MIT, active, ships compiled TextMate grammars). Used inside `/learn/*` MDX for `lang="asm-mips"` and `lang="boolean"` code blocks.

Custom grammars for MIPS asm + Boolean expression registered with shiki at build (extends `chevrotain` grammars from K-map parser).

### Markdown frontmatter parsing (MDX examples)

Locked pick: `gray-matter` (npm, MIT, active). Standard for Next/MDX content frontmatter.

### Schema-driven form generation (settings panel, K-map input)

Locked pick: `react-hook-form` + Zod resolver. Forms over Zod schemas. Both MIT, both active.

### Date/time

Native `Intl.DateTimeFormat`. No library import.

### Number formatting

Native `Intl.NumberFormat`. No library import.

### Animation engine

Locked picks already in STACK: `framer-motion` (for DOM motion) + `@react-spring/three` (for R3F springs). Both MIT, both active. No hand-roll.

### State machine (sim-engine)

Hand-roll justified — sim is pure-function `(state, action) → state'`, ≤ 100 LOC executor + step driver. XState is event/state-chart-shaped, overkill for monotonic cycle stepping; adapter cost > hand-roll cost. Documented per scan rationale.

### Object pool (Three.js Vector3/Quaternion/Matrix4)

Hand-roll justified — ~30 LOC per pool type. No mature OSS pool sized for this exact shape.

### Bits utilities (two's complement, sign-extend, base conversions)

Hand-roll justified — `~60 LOC`, no mature OSS replacement for this exact shape. `bitwise-buffer` exists but operates on buffers; ours operates on numbers + typed arrays.

### Snapshot tier-1 fragment encoding (base64url)

Native `btoa` / `atob` doesn't handle URL-safe base64 directly. Tiny wrapper (~10 LOC) hand-roll OR `uint8-to-base64` npm. Locked: hand-roll (~10 LOC), trivial threshold.

## CI gate

`tools/lint/oss-import-first.ts`:
- Greps `packages/*/src` + `apps/web/src` + `apps/backend/convex` for hand-rolled patterns that match libraries above (canonicalize, fuzzy search, worker pool, etc.)
- Asserts each hand-roll has matching exception entry in this ADR with rejection rationale
- Asserts each library above appears as a direct dep in the consuming package

## Audit refresh cadence

Quarterly. Re-scan npm + GitHub for new libraries in each category. Update ADR with newer-better choices when found + migrate.

Trigger immediate re-scan when:
- Hand-rolled component exceeds 100 LOC (signal that we're past trivial threshold)
- A locked library fails active-maintenance gate (6-month dep release gate)
- Operator pushback names a specific library we missed
