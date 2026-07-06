---
name: rayu-theme-factory
description: Generate cohesive, distinctive visual themes — a coordinated palette, typography, and mood — and emit them as ready-to-use tokens. Use when you need to produce one or more complete theme options for a UI.
---

# Rayu Theme Factory

Produce a *complete, coherent theme* (or a few options) from a brief: a palette, type pairing, and supporting tokens that feel designed together — then emit them as usable tokens. This is theme *generation*; for the token *architecture* (semantic layering, dark mode mechanics) use `rayu-design-system`.

## A theme is a coordinated set, not a random palette

Every theme has:
1. **A mood / direction** — one or two words (e.g. "warm editorial", "clinical precision", "retro arcade"). Decide this first; every choice serves it.
2. **A palette** — derived intentionally (below), not picked at random.
3. **A type pairing** — display + body that match the mood.
4. **Supporting tokens** — radius, shadow, spacing feel, motion character.

## Build the palette intentionally

Work in **HSL/OKLCH**, not raw hex, so relationships are controllable:

- Pick **one base hue** that carries the mood.
- Derive a **neutral ramp** (background → surface → border → muted text → text) by holding a slight hue tint and stepping lightness — neutrals with a hint of the brand hue feel cohesive; pure grey feels generic.
- Choose an **accent** via a deliberate relationship to the base: analogous (calm), complementary (energetic), or triadic (playful).
- Add **semantic states** (success/warning/danger/info) tuned to the palette, not stock bootstrap colors.
- Generate **tints/shades** by stepping lightness in a consistent ratio (e.g. 50→900).

Always check contrast for text-on-surface pairs (≥ 4.5:1 body) in *both* light and dark.

## Type pairing

Match faces to the mood: a high-contrast serif for editorial, a geometric sans for modern/tech, a grotesque for neutral/utility, a mono accent for data. Set the scale and weights (see `rayu-design-system` for the scale mechanics). Avoid the default "system-ui everywhere" unless the mood is deliberately plain.

## Emit usable tokens

Output the theme as something a build can consume directly:

```css
:root {
  /* mood: warm editorial */
  --bg:#fbf7f0; --surface:#f3ece0; --border:#e3d8c6;
  --text:#241c14; --text-muted:#6b5d4b;
  --accent:#b4531f; --accent-contrast:#fff;
  --success:#3f7d3f; --danger:#a23b2d;
  --radius:0.375rem; --shadow:0 1px 2px rgba(36,28,20,.12);
  --font-display:'Fraunces', serif; --font-body:'Inter', sans-serif;
}
```

Provide the matching **dark variant** by remapping the neutrals (don't just invert). If asked for multiple themes, give 2–3 *distinct* directions, each internally coherent — not three tints of the same idea.

## Distinctiveness check

Before shipping a theme, ask: would this same palette/type show up for any random brief? If yes, push the base hue, the neutral tint, or the type pairing toward the specific mood. (See `rayu-frontend-design` on avoiding the generic default.)

## Working in the rayu swarm

The `design` subagent uses this to generate the theme that goes into its PRD; the `frontend`/`mobile` collaborators implement the emitted tokens. Persist the chosen theme tokens to `.rayu/swarm/` so the build stays consistent.
