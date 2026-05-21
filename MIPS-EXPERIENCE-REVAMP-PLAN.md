# MIPS Experience Revamp — Detailed Plan

Status: APPROVED (operator + agent, this session). Build not started.
Scope: the MIPS datapath experience only (routes, focus sandbox, assembly editor, 2D/3D rendering). K-map / pipeline / learn are OUT of scope here.
Workflow contract: **fix → `bun run fix` → commit/push only. NO build/deploy** until operator explicitly says ship. Dev server (`next dev`, `reactStrictMode:false`) stays live for review. Every change keeps lintmax clean and the 199 ledger gates green; rebaseline visual snapshots when scenes change.

## Why
17 near-identical `/mips/[name]` routes are a leftover. The engine already drives the datapath + registers/memory from assembled code. Consolidate into one coherent, intuitive experience with a clear ladder: guided single-instruction study → free-form program writing → 2D/3D views.

## Locked context (do not violate)
- Single-cycle datapath only. NO pipelining in the datapath (pipeline registers / forwarding muxes are deferred per NON-GOALS). The `/pipeline` route stays a separate stage-time analyzer.
- IF/ID/EX/MEM/WB on the datapath are *logical phase highlights* of single-cycle, not real pipeline stages. Optionally clarify labeling so learners don't mistake it for pipelining.
- 17-instruction encodable floor: add addi and andi beq bne j lui lw nor or ori sll slt srl sub sw.
- Reuse existing engine — do NOT reinvent: `controlFor`, `executeStep`, `encodeInstruction`, `decodeInstruction`, `datapathValues`, `execution.ts` (createProgram seedable / current / stepForward / stepBack / runToCursor / runToBreakpoint / syscalls), `generated/topology.ts` (COMPONENTS + PATHS + roles), `generated/stepTraces.ts` (activePaths / componentsForPaths / STEPS), `asm-grammar.ts` (assemble), `criticalComponents`/`criticalPath`.
- lintmax discipline (CLAUDE.md): cn() only, arrow fns, exports at end, no `!`/`any`/ts-ignore, no `??`/`||` config fallback, no comments, file-level ignore headers at top.

## Route architecture
Three routes, one shared engine + one shared workspace renderer.

### `/mips` — landing (plain navigation, NOT a redirect)
- Categorized instruction picker (grid AND/OR dropdown), grouped: arithmetic / logical / memory / branch / jump. Doubles as a reference/cheat-sheet.
- A prominent "Write assembly →" entry/card → `/mips/assembly`.
- Picking an instruction navigates to `/mips/[name]`. No redirect indirection.

### `/mips/[name]` — FOCUS SANDBOX (structured controls, NO code editor)
The single biggest new piece. Pick one instruction, then tweak it via structured inputs (never free text — that is what keeps focus distinct from assembly):
- **Format-aware operand controls** (depend on instruction format):
  - R-type: `$rd`, `$rs`, `$rt` register dropdowns ($zero…$ra).
  - I-type: `$rt`, `$rs` dropdowns + **immediate** number field (16-bit range).
  - shift (sll/srl): `$rd`, `$rt` + **shamt** (5-bit).
  - branch (beq/bne): `$rs`, `$rt` + offset.
  - J (j): target field. lui: `$rt` + imm.
- **Source register value seeds** — editable values for the operand source registers (e.g. `$t1=10`, `$t2=3`) so the ALU computes real, visible results (not zeros). This is the point of "see the difference."
- Live, reactive on every change: control signals, active datapath path, registers/memory, machine-code hex.
- **Encoding-fields breakdown panel (first-class, the standout feature)**: show the 32-bit word split into `opcode | rs | rt | rd | shamt | funct` (or I/J layout) with each field's value, updating live as operands change. Connects assembly ↔ binary ↔ datapath in one view (change `$rd` → see rd bits flip AND RegDst mux/writeback wire change).
- **"open in assembly →"** link: drops the current instruction (with chosen operands) into `/mips/assembly`. The graduation bridge.
- NO Monaco editor here. Remove the editor drawer that currently exists on `/mips/[name]`.

### `/mips/assembly` — THE EDITOR (free-form programs)
Largely already built (current DatapathWorkspace editor). Owns Monaco. Free-form multi-instruction assembly → assemble live → run/step the program; registers/memory accumulate; example chips; instruction nav; keyboard; play=run-whole-program.

## Datapath rendering — 2D / 3D toggle (applies to focus + assembly)
- `[ 2D | 3D ]` segmented control, top of the inset. **2D is the default.** Persist choice in localStorage + URL (`?view=2d|3d`). Framing: 2D is the clearer/faster/accessible *better* default (instant, no WebGL, crisp labels), 3D is the immersive/cinematic mode — NOT "lite mode".
- **One id set, two geometries.** Component/path ids (frozen contract) are the single source. Add a **2D layout map** (`id → {x,y,w,h,shape,ports}`) alongside the existing 3D positions. Both renderers consume the same ids + same engine state.
- **2D renderer = data-driven SVG** (NOT a hand-authored static SVG, and NOT a port of the old ref/ StaticDatapathSvg). Redraw from data for maximum control:
  - Canonical left→right single-cycle layout (PC→IM→RegFile→ALU→DataMem→WB; control on top; sign-extend/shift below).
  - Wires as **orthogonal/Manhattan routed waypoints** (point arrays in data → themeable + animatable), not diagonal lines.
  - Theming: dark/light, per-type node colors consistent with 3D (registers blue, mux green, gates yellow, ALU red), luminance-contrast labels.
  - Animation: `stroke-dashoffset` marching-ants along active wires (2D twin of the 3D traveling pulse); node glow on active. Respect `prefers-reduced-motion` (freeze) for a11y + deterministic visual snapshots.
  - **2D-only pedagogy wins**: print the actual data value ON each active wire (e.g. `$t1=10` into ALU, `13` out) and **bus bit-widths** (32/5/16). Optional togglable "show all signal values" layer.
- 3D renderer = current cinematic scene (per-type colors, shadows, bloom, contact shadows, lightformer env, glued front-face labels, thick drei Line wires).
- Shared across both renderers + both pages: hover-reveal labels, click→inspect place-card, camera/focus, keyboard map, run/step, register/memory panel.

## Reference-while-editing (both surfaces, progressive)
- Selecting/hovering an instruction (dropdown, focus, or editor) surfaces: format (R/I/J fields), what it does, which control signals it asserts. Turns the surfaces into learning aids.

## The old 2D (ref/) — reference only, do NOT port
`~/mips/ref/src/features/datapath/` (StaticDatapathSvg + DatapathDiagram) is the prior hand-SVG version. Highlight language to learn from (red=active default path, yellow=modified path, blue/orange control signals, clickable hitboxes). We REDRAW data-driven instead of porting. Built on `core/mips/single-cycle/{control,diagram,highlight,inspector}` — same contract our topology froze against.

## Build sequencing
1. **`/mips/assembly` consolidation** — move the existing editor workspace to `/mips/assembly`; make `/mips` a real landing (picker grid + "Write assembly" card). Fastest win; reuses engine. Keep per-instruction URLs working.
2. **Focus sandbox + encoding breakdown** — `/mips/[name]` becomes structured operand controls + register-value seeds + live datapath/reg/mem/control + encoding-fields panel + "open in assembly". Remove its editor drawer.
3. **2D renderer + toggle** — data-driven SVG datapath (layout map + wire waypoints), `[2D|3D]` toggle (2D default, persisted), values-on-wires + bit-widths, shared state. Biggest new surface; do last.

## Done = every bullet above shipped, lintmax clean, 199 gates green at the live tree, visuals rebaselined, committed/pushed. Deploy only on operator's go.
