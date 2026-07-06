---
name: rayu-brand-guidelines
description: Define or apply a brand's identity system — logo usage, color, typography, voice, and imagery rules — so everything produced stays on-brand. Use when establishing a brand or enforcing consistency across deliverables.
---

# Rayu Brand Guidelines

A brand system is the rulebook that keeps every artifact recognizably the same author. Use this to *author* a brand guide from a brief, or to *apply* an existing one consistently.

## What a brand system contains

1. **Foundation** — mission/positioning in a sentence, and the personality in 3–5 adjectives (e.g. "precise, warm, confident"). Every other rule should trace back to these.
2. **Logo** — primary + secondary marks, clear-space (minimum padding around it), minimum size, and a do/don't list (don't stretch, recolor, rotate, add effects, place on busy backgrounds).
3. **Color** — primary, secondary, neutral ramp, and semantic colors, each with exact values (hex + the print/space equivalents if relevant). Specify which combinations are allowed and their contrast.
4. **Typography** — the brand typefaces (display, body, and a fallback stack), the type scale, and weight usage. Name when each is used.
5. **Voice & tone** — how the brand writes: person (we/you), formality, sentence length, words it uses and avoids, and how tone shifts by context (marketing vs error message vs docs).
6. **Imagery & iconography** — photography style, illustration style, icon set + stroke weight, and the overall visual texture.
7. **Layout** — grid, spacing rhythm, and composition tendencies (dense vs airy).

## Authoring a brand from a brief

- Derive everything from the **personality adjectives** — they're the throughline. "Playful" → rounder type, brighter accent, looser spacing; "authoritative" → sturdier serif, restrained palette, tighter grid.
- Make the color + type choices *specific* to the brand's world, not defaults (see `rayu-frontend-design` and `rayu-theme-factory`).
- Write the voice section with **concrete examples**: a sample headline, a sample button label, a sample error message — show the voice, don't just describe it.
- Output a structured guide (one markdown doc or a set of tokens) downstream agents can follow verbatim.

## Voice: make it usable

Give do/don't pairs, not adjectives alone:

```
Personality: confident, plain-spoken, never hype.
✓ "Your changes are saved."        ✗ "Awesome! We totally saved your stuff! 🎉"
✓ "Couldn't connect. Try again."   ✗ "Oops! Something went wrong :("
Use: clear verbs, sentence case, active voice.
Avoid: exclamation points, jargon, anthropomorphizing the product.
```

## Applying / auditing against a brand

When a guide exists, **conform to it** — don't reinvent. Audit deliverables against the rules: logo clear-space and min-size respected, only approved colors, correct typefaces and scale, voice consistent, imagery on-style. Flag each violation with the rule it breaks and the fix.

## Working in the rayu swarm

The `design` subagent can author the brand section of its PRD with this; collaborators apply it and keep tokens/voice consistent across UI, copy, and generated assets (pairs with `rayu-theme-factory` for color/type and `rayu-canvas-design` for on-brand graphics). Persist the brand rules to `.rayu/swarm/` so the whole swarm stays on-brand.
