---
name: rayu-web-testing
description: Approach for testing and verifying local web applications end-to-end — driving a real browser, finding selectors, capturing screenshots/logs, and stabilizing flaky tests. Use when verifying frontend behavior or writing E2E tests.
---

# Rayu Web Testing

Verify what the user actually sees by driving a real browser against the running app. Prefer a mainstream automation library (Playwright recommended; Cypress/Puppeteer are fine) and write small, deterministic scripts.

## Decision tree: pick the cheapest approach that works

```
Is the page static HTML (no JS rendering)?
├─ Yes → read the HTML directly, grab selectors, assert on the served markup.
└─ No (dynamic app) → is the dev server already running?
   ├─ No  → start it, wait until it's actually serving, THEN drive the browser.
   └─ Yes → reconnaissance-then-action:
            1) navigate, wait for network idle / a known element
            2) screenshot or dump the DOM to discover real selectors
            3) act using the discovered selectors
            4) assert on visible state, not internals
```

## Server lifecycle

Don't assume the server is up. Start it (`npm run dev`, `vite`, etc.), poll the port/URL until it responds, run the test, then tear it down. For multi-service apps (API + web), start each, wait for each to be healthy, then drive. Never hard-code `sleep` — wait on a real readiness signal (port open, health route 200, or an element present).

## Selectors: stable first

Priority order:
1. Role/label/text (`getByRole('button', { name: 'Save' })`, `getByLabel`) — closest to what users perceive.
2. `data-testid` hooks added intentionally for tests.
3. CSS/structure — last resort; brittle.

Avoid selectors tied to styling or DOM depth. If you can't find a stable hook, add a `data-testid` to the app rather than writing a fragile selector.

## Minimal Playwright shape

```js
import { test, expect } from '@playwright/test'

test('user can submit the contact form', async ({ page }) => {
  await page.goto('http://localhost:5173/contact')
  await page.getByLabel('Email').fill('a@example.com')
  await page.getByRole('button', { name: 'Send' }).click()
  await expect(page.getByText('Thanks — we got it')).toBeVisible()
})
```

## Evidence: screenshots + logs

Capture a screenshot on failure (and at key steps for UI work) — "a picture is worth a thousand tokens" when diagnosing. Collect browser console errors and failed network requests; they explain most "it just doesn't work" reports. Surface those in the result rather than guessing.

## Killing flakiness

- **Wait on conditions, not time.** Use auto-waiting assertions / `waitFor`, never fixed delays.
- **Isolate state.** Reset DB/storage between tests; don't depend on test order.
- **One reason to fail per test.** Small, focused specs localize breakage.
- **Control the network.** Mock third-party calls so a flaky upstream doesn't fail your suite.
- **Pin the viewport** for layout-sensitive assertions.

## Coverage priorities for a web app

1. Critical happy paths (auth, the primary conversion flow).
2. Form validation + error states.
3. Responsive behavior at one mobile + one desktop width.
4. Accessibility smoke (focus order, a labelled primary action).

## Working in the rayu swarm

The `review` subagent and `security` collaborator use this to verify produced UI against the spec, returning a concrete pass/fail + a Fix List. Keep scripts black-box and short; report high-signal results (what failed, the screenshot/log), not a play-by-play.
