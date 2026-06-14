# USERS

## Primary

Learners exploring low-level digital logic and computer architecture. Self-paced, self-directed. Reading textbooks, watching lectures, working through exercises elsewhere; come here to *see the thing*.

Concrete shapes:
- Building intuition for a MIPS instruction by stepping through its datapath animation
- Comparing two instructions side-by-side to see what changes
- Practicing K-map grouping on a truth table and self-checking against the solver
- Exploring how a 5-variable K-map's wraparound actually works (the headline 3D win)
- Deriving the Control unit truth table and seeing it minimized in K-map, then watching the same signals fire in the datapath

## Anonymous-only

Every visitor is anonymous; there are no accounts. Every feature works with no identity:
- Pick instruction, step, run, reset
- Author truth table, group cells, see PIs
- Save a snapshot to the local in-browser list
- Share via URL → recipient opens the link, sees the exact same sim state, no wall
- Side-by-side compare, pipeline view, critical path — all anonymous

## Local snapshots + sharing

- Snapshots live in the visitor's own browser (localStorage), surfaced as the `/me` local list
- Sharing carries the whole state in the URL fragment; the recipient decodes it client-side
- States too large for a URL fragment are non-shareable by design (tier `'oversize'`)

## Not addressed

- Accounts / login / cross-device persistence
- Real-time multiplayer collaboration
- Classroom / instructor dashboards / grading
- Curriculum-shaped lesson sequencing
- Per-user analytics surface

Listed for clarity; deferred items live in `NON-GOALS.md` with explicit trigger.

## Tone and copy

Audience is technical learners. Copy is direct, accurate, terse. No infantilizing, no gamification, no "great job!" toasts. Pedagogy is the product, not retention loops.

No reference to institution, course code, semester, or program in product surface, copy, docs, OG cards, metadata. Domain language only — "instruction", "control signal", "Boolean minimization", "prime implicant", "datapath".
