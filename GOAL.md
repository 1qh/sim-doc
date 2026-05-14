# GOAL

Ship world-class 3D interactive viz containing:

- **MIPS datapath** — single-cycle topology, every encodable instr animated, control signals shown, critical path overlay, side-by-side compare, pipeline stage-time + hazards
- **K-map** — 2D ≤4 vars / 3D toroidal ≥5 vars, all input modes, interactive grouping, PIs + EPIs + min SOP/POS, optional solver reveal
- Single app, anonymous-first, optional login, content-addressed permalinks per state

## World-class means

- Apple/NVIDIA silicon-reveal aesthetic tier
- 60 fps mid-tier laptop, WebGPU primary + WebGL fallback
- Keyboard-first, WCAG AA, reduced-motion respected
- Deterministic engine, frame-accurate replay
- Save/share permalink content-addressed edge-cacheable
- Substrate published OSS w/ foundation demos

## Runs on

Single MacBook compose stack = prod shape. Dokploy VM + Cloudflare bearer + Convex self-host = deploy. Migration laptop↔VM↔K8s = re-point IaC, never rewrite. See `DEPLOY.md`.

## Floor

Every item in `REQUIREMENTS.md` + every "more not less" ratchet. Out-of-scope items in `NON-GOALS.md` w/ explicit trigger.
