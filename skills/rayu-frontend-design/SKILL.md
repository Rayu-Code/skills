---
name: rayu-frontend-design
description: Guidance for building distinctive, intentional web/mobile UI that doesn't read as a templated default. Use when designing or reshaping a UI — choosing visual direction, type, layout, motion, and microcopy. Pairs with the design subagent's PRD.
---

# Rayu Frontend Design

Build interfaces a real design studio would ship: opinionated, specific to the brief, and free of the generic "AI default" look. Your job is to make deliberate choices you can justify — not to assemble safe defaults.

## 1. Ground the design in a real subject

Before any pixels, name three things and write them down:

- **Subject** — what the product actually is, in one concrete sentence.
- **Audience** — who uses it and what they care about.
- **The page's single job** — the one action or understanding the page must produce.

If the brief leaves these open, decide them and state your decision. Pull distinctive details from the subject's own world (its materials, vocabulary, artifacts). If a `.rayu/swarm/` design PRD or shared brief exists, treat it as the source of truth and build from it instead of re-deriving.

## 2. Avoid the generic default

Current "AI-generated" UI clusters around a few tells. Treat these as smells, not destinations:

- Cream/off-white background + high-contrast serif + a single terracotta accent.
- Near-black background + one acid-green or vermilion accent.
- Hairline-rule "broadsheet" layouts with zero radius and dense columns.
- Decorative `01 / 02 / 03` numbering applied to content that isn't actually a sequence.

These are fine *only* when the brief explicitly calls for them. Where the brief leaves an axis free, spend that freedom on a choice specific to the subject — not on a default.

## 3. Make the type carry the personality

- Set a real type scale (e.g. a modular scale around 1.2–1.333) with intentional weights and spacing — not browser defaults.
- Pair a characterful **display** face (used with restraint) with a clean **body** face; add a **utility/mono** face only if data/captions need it.
- Decide line-height per role (tight for display, ~1.5 for body) and constrain measure (~60–75ch for body text).

## 4. Structure should encode meaning

Eyebrows, dividers, numbering, and labels should reflect something true about the content, not decorate it. Ask "does this device carry information the reader needs?" before adding it. One memorable **signature element** per page; keep everything around it quiet.

## 5. Motion, deliberately

Choose where motion serves the subject: a page-load reveal, a scroll-triggered moment, hover micro-interactions. An orchestrated moment beats scattered effects. Keep durations in the 150–300ms range for UI feedback. **Always** honor `prefers-reduced-motion: reduce` — gate non-essential animation behind it.

## 6. The accessibility floor (non-negotiable)

- Contrast ≥ 4.5:1 for body text, ≥ 3:1 for large text and meaningful UI.
- Visible keyboard focus on every interactive element; logical tab order.
- Real semantic HTML (`button`, `nav`, `main`, headings in order); `alt` text; labels tied to inputs.
- Touch targets ≥ 44×44px with spacing.
- Responsive down to ~360px with no horizontal scroll.

## 7. Microcopy is design material

Write from the user's side of the screen: name things by what people control, not how the system is built. Active voice on controls ("Save changes", not "Submit"), and keep an action's name consistent through its flow (a "Publish" button → a "Published" toast). Make empty and error states do real work: say what happened and what to do next.

## 8. Process: plan → critique → build → critique

1. **Plan** a compact token system: 4–6 named hex colors, 2+ type roles, a layout concept (sketch it in prose/ASCII), and the one signature element.
2. **Critique the plan against the brief** — if any part reads like the default you'd produce for any similar page, change it and say why.
3. **Build** from the revised plan; derive every color/type decision from the tokens (see the `rayu-design-system` skill for the token model).
4. **Critique again** while building — verify the a11y floor, responsiveness, and reduced-motion. Cut one decoration that doesn't serve the brief.

## Working in the rayu swarm

When you're the `frontend` (or `mobile`) collaborator, implement against the `design` PRD and the backend contract from `.rayu/swarm/`. Use the `Skill` tool to also pull `rayu-design-system` for the token model. Report concisely; persist durable design decisions to your domain section.
