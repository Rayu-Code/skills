---
name: rayu-design-system
description: Build and apply a consistent design-token system — color, spacing, type, radius, shadow — with theming, dark mode, and responsive scales. Use when setting up a UI's foundation or enforcing visual consistency across components.
---

# Rayu Design System

A design system is the small set of decisions every component obeys. Define the tokens once, name them by intent, and never hard-code a raw value in a component again.

## 1. Two layers of tokens

- **Primitive tokens** — the raw palette/scale: `color-blue-600: #2563eb`, `space-4: 1rem`, `font-size-lg: 1.125rem`. No meaning, just values.
- **Semantic tokens** — intent mapped onto primitives: `color-bg`, `color-surface`, `color-text`, `color-text-muted`, `color-border`, `color-accent`, `color-danger`. **Components consume only semantic tokens.** This is what makes theming and dark mode a one-file change.

Expose them as CSS custom properties (or your framework's theme object):

```css
:root {
  --color-bg: #ffffff;
  --color-surface: #f7f7f8;
  --color-text: #18181b;
  --color-text-muted: #52525b;
  --color-border: #e4e4e7;
  --color-accent: #2563eb;
  --radius-md: 0.5rem;
  --space-2: 0.5rem; --space-4: 1rem; --space-8: 2rem;
}
```

## 2. The scales

- **Spacing** — one scale, geometric or 4px-based (`2,4,8,12,16,24,32,48,64`). Every margin/padding/gap comes from it.
- **Type** — a modular scale (ratio ~1.2–1.333) with defined roles (display, h1–h3, body, caption), each with weight + line-height. Body ≈ 16px / 1.5.
- **Radius / shadow / border** — 2–4 steps each. Resist per-component one-offs.
- **Z-index** — a named ladder (`dropdown, sticky, overlay, modal, toast`) so stacking is intentional.

## 3. Theming and dark mode

Because components use semantic tokens, a theme is just a different mapping of semantic → primitive. Implement dark mode by overriding the semantic layer:

```css
:root[data-theme='dark'] {
  --color-bg: #0b0b0e;
  --color-surface: #18181b;
  --color-text: #fafafa;
  --color-text-muted: #a1a1aa;
  --color-border: #27272a;
}
```

Verify **both** themes hit the contrast floor (≥ 4.5:1 body). Respect `prefers-color-scheme` for the default and let users override.

## 4. Responsive scale

Mobile-first. Define a few breakpoints (`sm 640, md 768, lg 1024, xl 1280`) and a fluid type/space step if you want headings to scale (`clamp()` is your friend). Containers get a max-width + auto margins; never let content run edge-to-edge on wide screens.

## 5. Component states

Every interactive component defines all of: `default, hover, active, focus-visible, disabled`, plus `loading` and `error` where relevant. Focus-visible must be clearly visible (don't remove outlines without replacing them). Document the states next to the component.

## 6. Consistency checklist

- [ ] No raw hex / px in components — only tokens.
- [ ] One spacing scale, one type scale, used everywhere.
- [ ] Semantic tokens only in components; primitives only in the theme map.
- [ ] Light + dark both pass contrast.
- [ ] All states defined incl. focus-visible.
- [ ] Icons from one set; consistent stroke/size.

## Working in the rayu swarm

The `design` subagent should emit these tokens in its PRD; the `frontend`/`mobile` collaborators implement against them (pair with the `rayu-frontend-design` skill). Persist the final token set to your `.rayu/swarm/` domain section so the whole swarm stays visually aligned.
