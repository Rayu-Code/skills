---
name: rayu-algorithmic-art
description: Create generative / algorithmic art with code — procedural patterns, flow fields, particle systems, fractals, and creative coding (Canvas, SVG, p5.js, shaders). Use when producing generative visuals, backgrounds, or animations from algorithms.
---

# Rayu Algorithmic Art

Make visuals from rules: a few parameters and an algorithm that produces something rich and often surprising. Use for generative backgrounds, abstract art, data-driven visuals, and ambient animation.

## Pick the medium

- **Canvas 2D** — raster, fast for thousands of particles/strokes. Best general default.
- **SVG** — vector, crisp + editable, great for plotter-style line art and scalable patterns (watch node counts).
- **p5.js** — friendly creative-coding API (sketch `setup()`/`draw()`); good for quick exploration.
- **WebGL / GLSL shaders** — per-pixel, GPU-fast; for fields, noise, fractals, and heavy real-time effects.

## The generative toolkit

- **Randomness, controlled.** Use a **seeded** PRNG (e.g. mulberry32) so a result is reproducible — store the seed. Prefer *distributions* over flat `Math.random()`: gaussian for natural clustering, weighted choices for variety.
- **Noise, not random, for organic form.** Perlin/Simplex noise gives smooth, coherent variation — the backbone of flow fields, terrain, clouds, and organic motion.
- **Flow fields:** sample a noise field for an angle at each point; move particles along the angle, leaving trails. A handful of lines yields complex, elegant results.
- **Particle systems:** each particle has position/velocity/lifespan; apply forces (noise, gravity, attraction); fade trails by drawing a translucent bg each frame.
- **Tiling & symmetry:** rotate/mirror around a center, or tile a motif with jitter, for pattern and mandalas.
- **Fractals / recursion:** subdivision, L-systems, and recursive shapes for self-similar structure.

## Composition still matters

Algorithmic ≠ chaotic. Apply real design judgment:
- **Constrain the palette** (3–5 colors from a coherent scheme — pair with `rayu-theme-factory`). Random rainbow output reads as noise.
- **Density and negative space** — let the piece breathe; a clear focal area beats uniform busyness.
- **Layering** — background field + mid-detail + sparse foreground accents.
- **Curate.** Generate many seeds, keep the best. The selection is part of the artistry.

## Minimal flow-field sketch (Canvas)

```js
const rand = mulberry32(seed)
const angle = (x, y) => noise(x * 0.002, y * 0.002) * Math.PI * 2
ctx.fillStyle = '#0b0b0e'; ctx.fillRect(0, 0, W, H)
ctx.globalAlpha = 0.06; ctx.strokeStyle = palette[0]
for (let i = 0; i < 1500; i++) {
  let x = rand() * W, y = rand() * H
  ctx.beginPath(); ctx.moveTo(x, y)
  for (let s = 0; s < 60; s++) { const a = angle(x, y); x += Math.cos(a) * 2; y += Math.sin(a) * 2; ctx.lineTo(x, y) }
  ctx.stroke()
}
```

## Animation & export

- Animate with `requestAnimationFrame`; advance a time/`z`-noise dimension for smooth evolution. Honor `prefers-reduced-motion` if it ships in a UI.
- Export stills via canvas `toBuffer`/`toDataURL` (PNG); for loops, capture frames and assemble (gif/webm) or ship the live sketch as a `rayu-web-artifacts-builder` artifact.
- Render at 2×+ for crisp stills.

## Working in the rayu swarm

The `asset-generation` subagent uses this for generative backgrounds/visuals; pair with `rayu-theme-factory` for the palette and `rayu-canvas-design` for finishing static compositions. Save outputs to the project asset path and report seeds so results are reproducible.
