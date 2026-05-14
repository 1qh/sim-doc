# ACCESSIBILITY-STATEMENT

Public accessibility statement. Renders at `/accessibility`. Per `A11Y.md` engineering floor.

---

## Commitment

This tool targets WCAG 2.2 Level AA on every interactive surface. Keyboard-first navigation, full screen-reader support, color-blind palettes, reduced-motion alternates. Every release runs accessibility tests; violations fail CI before deploy.

## What works

### Keyboard navigation

Every action invocable without mouse. Per-route keyboard matrix documented in the in-app `?` overlay. Cmd+K opens a global command palette searching everything.

### Screen readers

3D scenes carry DOM proxies with ARIA labels describing every active component, every signal value, every cell state. Live regions announce step changes ("Step IF complete. PC = 0x...").

Tested with NVDA (Windows), VoiceOver (macOS), VoiceOver (iOS).

### Color contrast

All body text ≥ 7:1 contrast against background (AAA tier). UI chrome ≥ 4.5:1. Color is never the sole carrier of meaning — hazards are amber + icon + label.

### Color-blind variants

Three alternate palettes — deuteranopia, protanopia, tritanopia — selectable in settings. Each shifts the accent + warning colors to maintain perceptual distinguishability.

### Reduced motion

`prefers-reduced-motion: reduce` collapses every animation to an instantaneous state swap. 3D camera transitions become cuts. Signal pulses become static highlights. No information lost.

User can also override the system preference in settings.

### Focus visible

Custom 2px focus ring on every focusable element. Never removed.

## What's a known limitation

### Mobile interaction

Below ~1024px viewport, the editor and drag-to-group K-map interactions are not enabled. Mobile gets full read-only access to learn pages, shared permalinks, and saved snapshots. Per `adr/mobile-read-only.md`.

We're tracking trigger conditions to flip mobile into full interactivity — multi-touch K-map grouping needs to prove cleanly designable first.

### Older browsers

Latest stable Chrome / Edge / Firefox / Safari only. Older browsers see a "current browser required" page. Per `BROWSER-SUPPORT.md`.

## Reporting issues

Spot an accessibility gap? Report via the disclosure address in `SECURITY.md` — we treat a11y bugs at the same priority as security bugs.

## Audit cadence

`axe-core` runs on every PR. `pa11y` runs nightly. NVDA + VoiceOver scripted walkthroughs run before each release. Manual audit quarterly.

## Conformance level

WCAG 2.2 Level AA targeted. Some AAA items met (contrast on body text). No claim of AAA conformance overall.
