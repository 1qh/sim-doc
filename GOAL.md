# GOAL

Ship a world-class 3D interactive visualizer that contains:

- **MIPS datapath visualizer** — single-cycle topology, every encodable instruction animated step-by-step with active components lit, control signals shown, critical path overlay, side-by-side instruction comparison, pipeline stage-time diagram with hazard detection.
- **Karnaugh map tool** — 2D for ≤4 vars, 3D for ≥5 vars, truth table / Boolean expression / minterm-list / maxterm-list input, interactive grouping, automatic prime implicants + essential prime implicants + minimal SOP + minimal POS, optional solver reveal.
- **Single pure-client web app**, anonymous-only with no accounts, every sim state shareable by a self-contained URL (state encoded in the URL fragment, decoded client-side).

## What "world-class" means here

- Visual aesthetic at Apple/NVIDIA silicon-reveal tier — see `UX-DOCTRINE.md`
- 60 fps on mid-tier laptop hardware, WebGPU primary path with WebGL fallback
- 100% keyboard-driven (every action invocable without mouse)
- WCAG AA accessibility floor, reduced-motion respected
- Deterministic sim engine — same input produces frame-accurate same animation
- Share any sim state by a self-contained URL; states too large for a URL fragment are non-shareable by design
- Substrate published OSS, foundation-app demonstrates every substrate primitive generically

## What it must run

- Operator zoo runs on a single MacBook through the same compose stack as production
- Production deploys the static client app to a VM with Cloudflare DNS + edge cache
- Migration laptop ↔ on-prem ↔ cloud-VM is re-point IaC + redeploy, never rewrite

## What ships in the locked floor

Every feature in `REQUIREMENTS.md` and every "more not less" ratchet on top. No phased carve-outs. Items genuinely outside scope live in `NON-GOALS.md` with explicit trigger.
