# early-velocity-mode

## Decision

Velocity-mode policy active. Trades pre-merge CI safety for shipping speed during bootstrap + foundation + initial product feature phases. Per operator instruction; cited per `book/HARD-RULES.md` exception for explicit operator-requested workflow.

## What velocity-mode means

| Aspect | Velocity-mode state |
|---|---|
| GitHub Actions | Disabled (Actions workflows present in `.github/workflows/` but trigger conditions force-skipped via `if: false` or absent until activation trigger) |
| Pre-commit hook (`sh up.sh && git add -u`) | Wired but optional — `git commit --no-verify` is allowed |
| `bun run fix` (lintmax) | MUST pass locally before every commit. Author runs it manually. `--no-verify` is a velocity shortcut, NOT a quality bypass. |
| Pre-push hook | Disabled / absent |
| Branch protection on GitHub | Off |
| PR required | Off — direct push to `main` is allowed for solo-operator velocity |
| Commit frequency | Maximum — smallest task or bug fix is one commit. No batching, no waiting for "logical chunk" |
| Branch model | Trunk-based on `main`, per `book/HARD-RULES.md` |

## Pre-commit ritual (manual, until pre-commit hook re-enables)

```bash
bun run fix      # lintmax full pass, must exit 0
git add -A       # or specific paths
git commit --no-verify -m "..."
git push
```

If `bun run fix` doesn't pass: fix the lint failure first. Never commit lint-red.

## Why velocity-mode exists

Bootstrap phase has frequent restructure + scaffolding + many sub-1-commit fixes. Pre-commit + Actions + PR review on every push adds 30-60s × N commits = hours wasted on a solo loop. Velocity-mode trades pre-merge safety for iteration speed during a window where the author IS the reviewer and the loop IS the safety net.

Per `book/PHILOSOPHY.md` "Stopping is expensive" — workflow friction is a class of stopping. Velocity-mode minimizes friction.

## Trigger to exit velocity-mode

Velocity-mode exits when ANY of these conditions fires:

1. **Phase 18 reached** per `adr/foundation-bootstrap-order.md` (polish ratchet phase — by then the product is feature-complete and pre-merge gates start paying for themselves)
2. **First external contributor** opens a PR (PR review depends on CI green)
3. **First public traffic** beyond operator + close collaborators (real users mean blast radius increases)
4. **Operator explicitly declares** "velocity-mode off" in a session
5. **Repeated lint regression** caught post-commit ≥3 times in a single week (signal that author-side discipline is failing; bring back the pre-commit hook)

## What ratchets up on exit

When velocity-mode exits:

| Layer | Activated |
|---|---|
| GitHub Actions | Workflows re-enabled (`if:` conditions removed); CI gates per `VERIFY.md` enforced |
| Pre-commit hook | `simple-git-hooks` re-armed, runs `sh up.sh && git add -u` on every commit |
| Pre-push hook | Lightweight push-time gate added (build + type check at minimum) |
| Branch protection | Enabled on `main` — required status checks per `book/HARD-RULES.md` |
| PR required | Discouraged-but-allowed direct push → required PR for substrate changes |
| Velocity-mode ADR | This ADR amended to reflect exit + rationale |

## Discipline mandates that DON'T relax under velocity-mode

These rules stay on regardless of velocity-mode:

- `bun run fix` (lintmax) must pass before commit. Per operator's explicit instruction.
- No commit of secrets (per `book/HARD-RULES.md` single-secrets-root).
- No deletion of work without commit history preserving it.
- No force-push to `main` without explicit operator approval.
- Conventional commit prefix on subject line.
- No AI / Claude / coauthor mentions in commit messages.
- Atemporal docs (no "we switched from", "previously", etc.).
- No school refs in product surface.

## Caught by

- `tools/lint/velocity-mode-marker.ts` (when written) — asserts `.github/workflows/*.yml` files carry the disable marker; flags removal that doesn't trip the exit-trigger ratchet.
- Author discipline — `bun run fix` exit code before every commit.
- Operator pushback if a commit lands lint-red.
- Periodic operator review of commit log for discipline regressions.
