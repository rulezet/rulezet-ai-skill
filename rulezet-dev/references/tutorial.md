# Tutorial (guided tour) — `app/features/tour/`

The graduation-cap icon next to the settings gear, on every page:

- **Tour of this page** — the page's hand-written steps (setting `tour.page.<endpoint>`), then
  everything else read from the page itself (sections, tabs, form parts, tables, actions).
- **Full tour of Rulezet** — setting `tour.full`: page after page, the whole page first, then section
  by section (the page's own steps without the button-level ones, `"level": "detail"`).

Steps are filtered by role **on the server** (`tour_core.py`): a visitor never receives admin steps.

## A step

```json
{ "target": "[data-tour=\"rule-content\"]", "title": "The rule", "text": "…",
  "roles": ["user"], "open": "[data-tour=\"rl-filters-btn\"]", "demo": "rule", "level": "detail" }
```

- `target`: CSS selector (missing → the step is skipped; `"@page"` = the whole page; none = centred).
- `text`: plain text — blank line = paragraph, `**bold**`, `` `code` `` (escaped, never HTML).
- `roles`: `anonymous`, `user` (any logged-in), `member` (logged in, not admin), `admin`, `perm:<key>`.
- `open`: clicked first when the target is hidden (filter box, tab). Collapsed `.h-card`s, Bootstrap
  collapses and `<details>` are opened automatically.
- `demo`: `rule` / `bundle` — the example (setting `tour.demo`) shown when the target is missing.
- `href`: the Next button goes there (ends the tour).

## Adding a page to the tutorial

1. Put `data-tour="<page>-<part>"` on the key elements of the template (on a real element, not a
   `<template>` or an `<option>`).
2. Write `app/config/tour.page.<endpoint>.json` (`{"title", "steps"}`) — endpoint as Flask names it,
   e.g. `tour.page.rule.rules_list`. Aliases (two endpoints, one page) in `ENDPOINT_ALIASES`.
3. Optionally add the page to `tour.full` with an `overview`.

Other settings: `tour.enabled`, `tour.roles` (first step: who you are browsing as), `tour.demo`.
Demo pages `/tour/demo/rule|bundle`, welcome page `/tour/welcome` (empty instance).
Engine: `static/js/components/tour/Tour.js`; launcher: `static/js/tour/tourLauncher.js`.
