# STYLING

CSS / styling layer + conventions. Lock per `adr/styling-css.md`.

## Stack

- **Tailwind CSS latest** (Tailwind 4 with the Lightning CSS engine and CSS-first config) — utility-first, JIT-compiled, design-tokens-driven
- **CSS Modules** for component-scoped styles where utilities are insufficient (complex state-driven styles, animations not expressible as utilities)
- **No CSS-in-JS** (no Emotion, styled-components, stitches) — runtime cost, hydration cost, no SSR-friendly default

Why Tailwind 4:
- Utility-first matches design-system component composition
- JIT compile keeps bundle minimal
- CSS-first config integrates with `packages/design-tokens` via CSS variables
- Active maintenance + latest-only per `book/PHILOSOPHY.md`
- No runtime cost
- Matches operator's existing pm4ai-standard stack

Why CSS Modules as escape valve:
- For animations with multiple keyframes
- For complex pseudo-class state machines
- For 3D-overlay positioning where utility classes get unreadable

## Design-tokens integration

`packages/design-tokens` emits:
- `tokens.css` — every token as CSS variable (`--color-bg-primary`, `--spacing-md`, etc.)
- `tailwind.preset.ts` — Tailwind preset that maps utility classes to those variables

App imports `tokens.css` at root layout. Every utility class resolves to a token variable; never hard-coded values.

## Conventions

### Allowed

- Tailwind utilities for layout, spacing, color, typography, sizing
- CSS Modules for component-scoped complex styles (file naming: `Component.module.css`)
- CSS-variable references (`var(--color-bg-primary)`) inside Module CSS
- `clsx` or `tw-merge` for conditional class composition
- `cva` (class-variance-authority) for component variants

### Banned

- Inline `style={{}}` (use utilities)
- Hard-coded hex / rgb / px values (use tokens)
- `important!` modifier (use specificity correctly)
- Global CSS overrides (use CSS Modules)
- Tailwind `arbitrary-values` (`bg-[#fff]`) — defeats token system
- Magic numbers in spacing (use spacing scale)
- CSS-in-JS runtime libraries

### Class composition pattern

```tsx
import { cva } from 'class-variance-authority';
import clsx from 'clsx';

const button = cva('inline-flex items-center justify-center rounded-md', {
  variants: {
    variant: {
      primary: 'bg-accent text-accent-fg',
      secondary: 'border border-fg text-fg',
      ghost: 'text-fg hover:bg-bg-elevated',
    },
    size: {
      sm: 'h-8 px-3 text-sm',
      md: 'h-10 px-4 text-base',
      lg: 'h-12 px-5 text-lg',
    },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});
```

## Color-blind palette switching

`packages/design-tokens` ships three alternate palette files. Root layout reads user preference and conditionally imports:

```tsx
<html data-palette={paletteVariant}>
```

Tailwind preset has `data-palette-deuteranopia:`, `data-palette-protanopia:`, `data-palette-tritanopia:` variants. Per `A11Y.md`.

## Dark mode

Single near-black palette by default. Light mode NOT in floor — per `UX-DOCTRINE.md` industrial aesthetic. ADR amendment required to add light mode.

## Reduced motion

`@media (prefers-reduced-motion: reduce)` block in `tokens.css` overrides every duration token to `0ms` and every easing to `linear`. Components consume tokens, never hand-typed durations; reduced-motion compliance is automatic.

## CSS budget

- Total CSS shipped to client < 30 KB gzip per route (per `PERFORMANCE.md`)
- Tailwind PurgeCSS at build removes unused utilities
- No font-loading FOUT (preload + `font-display: optional` for Latin subset)

## Caught by

- `tools/lint/no-inline-styles.ts` greps for `style={{` in JSX
- `tools/lint/no-arbitrary-tailwind.ts` greps for Tailwind arbitrary-value syntax
- `tools/lint/tokens-only.ts` asserts no hex / rgb / px-as-spacing literals in CSS Modules
- CSS bundle-size CI gate per `adr/perf-budget.md`
