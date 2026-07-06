---
name: rayu-web-artifacts-builder
description: Build self-contained, single-file interactive web artifacts — a demo, calculator, visualization, mini-tool, or prototype that runs by opening one HTML file. Use when the deliverable is a standalone interactive page, not a full app.
---

# Rayu Web Artifacts Builder

An "artifact" is a single, self-contained interactive page that works by opening one file — no build step, no server, no install. Think: a unit converter, a data visualization, an interactive explainer, a game prototype, a configurator.

## The contract

- **One file when possible.** Inline CSS in `<style>` and JS in `<script>`; embed small data as a JS object. The user can save it and double-click it.
- **No build, no network by default.** It must run offline from `file://`. If you need a library, pull it from a CDN with an integrity hash and a graceful fallback — but prefer vanilla JS / the platform.
- **Self-documenting.** A title, a one-line purpose, and obvious controls. No README required to use it.

## Skeleton

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Compound Interest Explorer</title>
  <style>
    :root { --bg:#0b0b0e; --fg:#fafafa; --accent:#22d3ee; }
    * { box-sizing: border-box }
    body { margin:0; font:16px/1.5 system-ui, sans-serif; background:var(--bg); color:var(--fg); }
    main { max-width: 720px; margin: 0 auto; padding: 2rem; }
    label { display:block; margin:.75rem 0 .25rem }
    input, button { font:inherit; padding:.5rem .75rem; border-radius:.5rem; }
    output { font-size:2rem; font-weight:700; color:var(--accent) }
  </style>
</head>
<body>
  <main>
    <h1>Compound Interest Explorer</h1>
    <label for="p">Principal</label>
    <input id="p" type="number" value="1000" />
    <output id="result">—</output>
  </main>
  <script>
    const p = document.getElementById('p')
    const result = document.getElementById('result')
    const fmt = new Intl.NumberFormat('en', { style:'currency', currency:'USD' })
    function update() { result.textContent = fmt.format(Number(p.value) * 1.05) }
    p.addEventListener('input', update); update()
  </script>
</body>
</html>
```

## Build it well

- **State → render.** Keep a small state object; write one `render()` that reflects state into the DOM; event handlers mutate state then call `render()`. Avoid scattered DOM tweaks.
- **Reach for the platform:** `<input type=range>`, `<details>`, `<dialog>`, `Intl`, `requestAnimationFrame`, `<canvas>`/SVG for graphics, `localStorage` for persistence.
- **Make it responsive and keyboard-usable** even though it's "just" an artifact — labels on inputs, visible focus, works at mobile widths.
- **Polish the empty/initial state** so it's obvious what to do.
- **Performance:** debounce expensive recompute; for animation use `requestAnimationFrame`, not `setInterval`.

## When NOT to use this

If it needs auth, a backend, multiple routes, or a real database, it's an app — use the `frontend`/`backend` collaborators and a real project scaffold instead. Artifacts are for one focused, client-only interaction.

## Verify

Open the file (or load its markup) and exercise the controls: inputs update output, no console errors, works at a narrow width. Ship the single file.

## Working in the rayu swarm

Use for quick interactive deliverables and prototypes. For visual styling depth pair with `rayu-frontend-design`; for generative graphics inside the artifact see `rayu-algorithmic-art`/`rayu-canvas-design`.
