---
name: playwright
description: Use Playwright for autonomous web-app checks: launch or target an app, exercise real browser behavior, debug failures, and decide whether to write a repeatable test or use interactive exploration.
license: MIT
compatibility: opencode
metadata:
  category: testing
  triggers: playwright, browser test, e2e, UI test, web app, page automation
---

# Playwright

Use Playwright when the task needs real browser behavior, not just static code inspection.

## Guidance

- Prefer the repo's existing Playwright setup if present.
- For one-off investigation, an ad-hoc Playwright script is fine if it gives fast, repeatable evidence.
- For regression coverage, turn the scenario into a normal Playwright test.
- Use accessible locators first: role, label, visible text, stable test ids.
- Avoid sleeps; wait for visible UI state, navigation, or specific responses.
- On failure, capture the useful evidence: error text, console errors, failed requests, screenshot or trace.

## MCP

Do not add a Playwright MCP by default.

Use MCP only when it is already configured or the task is mostly interactive exploration. Even then, convert durable findings into a script, test, or clear reproduction steps.
