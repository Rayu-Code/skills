---
name: rayu-canvas-design
description: Compose static visual graphics on an HTML5 canvas — posters, social cards, diagrams, OG images, certificates. Use when generating a designed raster/vector image programmatically rather than a live UI.
---

# Rayu Canvas Design

Use the `<canvas>` 2D context (or a headless equivalent like `node-canvas`/`@napi-rs/canvas`/`skia-canvas`) to compose a finished image: an Open Graph card, a poster, a quote graphic, a diagram, a certificate. This is for *generated artwork*, not interactive UI.

## Set up the surface

- Decide dimensions up front from the use case: OG image 1200×630, square social 1080×1080, story 1080×1920, A4 poster at 96dpi ≈ 794×1123 (×2–3 for print crispness).
- **Render at 2× and scale down** for crisp text/edges, or set `ctx.scale(dpr, dpr)` with a backing store at `width*dpr`.
- Establish a coordinate plan and a margin/safe-area before drawing.

```js
const W = 1200, H = 630, S = 2
const canvas = createCanvas(W * S, H * S)
const ctx = canvas.getContext('2d')
ctx.scale(S, S)
```

## Composition principles

- **Background first, foreground last.** Fill or gradient the bg, then layer shapes, then text on top.
- **One focal point.** A single dominant element (a headline, a logo, a number) anchors the eye; everything else supports it.
- **Grid + margins.** Keep content inside a safe margin (≈ 6–8% of width). Align to an invisible grid; don't eyeball every element.
- **Type hierarchy.** 2–3 sizes max. Measure text (`ctx.measureText`) to wrap, center, and avoid overflow — never assume it fits.
- **Contrast.** If text sits on an image, add a scrim (semi-opaque overlay) or shadow so it stays legible.

## Core 2D recipes

```js
// Gradient background
const g = ctx.createLinearGradient(0, 0, W, H)
g.addColorStop(0, '#0f172a'); g.addColorStop(1, '#1e3a8a')
ctx.fillStyle = g; ctx.fillRect(0, 0, W, H)

// Rounded rect (card)
ctx.beginPath(); ctx.roundRect(80, 80, W - 160, H - 160, 24)
ctx.fillStyle = 'rgba(255,255,255,0.06)'; ctx.fill()

// Text with manual wrap
ctx.font = '700 64px Inter'; ctx.fillStyle = '#fff'
ctx.textBaseline = 'top'
wrapText(ctx, headline, 120, 140, W - 240, 76)
```

Helper: split words, accumulate until `measureText(line).width > maxWidth`, then break — write a small `wrapText(ctx, text, x, y, maxWidth, lineHeight)`.

## Fonts and images

- **Register fonts** before drawing (headless libs need `registerFont`/`GlobalFonts.register`); a missing font silently falls back and ruins the layout.
- **Await image loads** before `drawImage`; use `drawImage(img, sx,sy,sw,sh, dx,dy,dw,dh)` for crop-to-fit (cover/contain math).
- Draw vector shapes for icons/logos when possible (crisp at any scale).

## Export

```js
// PNG (lossless, transparency) — best for graphics with text
const png = canvas.toBuffer('image/png')
// JPEG (smaller, no transparency) — photographic backgrounds
const jpg = canvas.toBuffer('image/jpeg', { quality: 0.9 })
```

Verify the output opens and the text isn't clipped before reporting done.

## Pitfalls

- Forgetting `textBaseline` → vertical misalignment.
- Not measuring text → overflow / clipping.
- Unregistered fonts → wrong typeface.
- Drawing before image/font load resolves → blank or fallback.
- Low resolution → blurry text; render at 2×+.

## Working in the rayu swarm

Use for generated image deliverables (OG/social cards, posters, diagrams). For *animated* or *interactive* visuals see `rayu-algorithmic-art`; for live product UI see `rayu-frontend-design`. Save assets to the project's image path and report each file with its dimensions.
