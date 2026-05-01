## Quick repository summary

- Single-file marketing/demo site: the entire app is in `main.html` (HTML + CSS + JS inline). There is no build system, package manager, or test harness present.
- Purpose: interactive landing page + live demo simulation (custom cursor, modal, live log feed, threat heatmap, risk calculator).

## High-level architecture (what to know first)

- All UI, styles, and behavior live inside `main.html`. Treat it as the single source of truth.
- Visual/UI layer: CSS variables in `:root` (colors, accents). Responsive breakpoint at 768px.
- Interaction layer: inline JS at bottom of `main.html`. Key domains:
  - Cursor: elements with ids `cursor` and `cursorRing` implement a custom cursor. Note `body { cursor: none }` disables the OS cursor.
  - Modal: `openModal()` toggles class `open` on element `#modalOverlay`.
  - Live feed: simulated events are pushed to `#logFeed` by the `addLog()` routine (uses `logMessages` array).
  - Threat heatmap: `#threatGrid` is populated with 200 `.threat-cell` elements; `updateGrid()` randomly assigns `.hot/.warm/.cool` classes.
  - Calculator: sliders (`#empRange`, `#recRange`, `#indRange`, `#matRange`) drive `updateCalc()` using arrays `industries`, `industryMultipliers`, `maturityMultipliers`.
  - Forms: the modal signup posts to Formspree (URL: `https://formspree.io/f/mlgaryad`), while `downloadWP()` only toggles a success message (no backend call).

## Critical hotspots / patterns to preserve

- DOM ids and selectors are relied upon extensively. When refactoring, preserve or update these ids and the JS that references them:
  - `cursor`, `cursorRing`
  - `modalOverlay`, `modalClose`, `modalForm`, `successMsg`, `formSubmit`
  - `logFeed`, `threatGrid`, `threatCount`, `alertBadge`, `packetCount`, `blockedCount`
  - `empRange`, `recRange`, `indRange`, `matRange`, `calcResult`, `calcSaving`
  - `wpEmail`, `wpSuccess`

- `logMessages`, `industries`, `industryMultipliers`, `maturityMultipliers` are paired arrays in JS — update both in lockstep if you add/remove industry options.
- The stat counter animation reads `data-target` attributes on elements with `[data-target]` — if you add new stat items, use the same attribute pattern and matching numeric formatting.

## Known quirks & gotchas (save time debugging)

- Cursor-less editing: `body { cursor: none }` plus `button, a, input { cursor: none }` are deliberate UI choices. Developers editing the page may find pointer feedback confusing. Use a temporary override (e.g., remove `cursor:none`) while developing.
- Mouse-enter transform concatenation: current code uses `cursor.style.transform += ' scale(1.5)'` on `mouseenter`. This mutates a string repeatedly and can produce invalid transforms if run multiple times — prefer setting a stable transform value instead of concatenation.
- Modal form submit: `document.querySelector('.form-input')` picks the first `.form-input` (work email). If you add more inputs, the submit handler will still only post that first value.
- `downloadWP()` only validates an email client-side and toggles `#wpSuccess` — no network call.
- Numeric equality checks in animations rely on exact floats (e.g., comparing `target === 0.8`) — be careful when modifying `data-target` values.

## How to run / preview locally (developer flows)

- Quick preview (open directly):
  - macOS (zsh): open `main.html` in the default browser: `open main.html`.
- Static server (recommended for local dev because some features work better over HTTP):
  - Python 3: `python3 -m http.server 8000` then open `http://localhost:8000/main.html`.
  - Node (if you prefer): `npx http-server -c-1` or `npx serve`.
- VS Code: use the Live Server extension to get live-reload while editing `main.html`.

## Editing guidance & examples

- Adding a new log message: update the `logMessages` array in `main.html`. Example shape: `{ type: 'ok'|'warn'|'danger', msg: 'Your message' }`.
- Changing the calculator industry options:
  - Edit both `industries` and `industryMultipliers` arrays in the JS section so indices remain aligned.
  - The slider `#indRange` currently maps 1..5 to array indices 0..4.
- Expanding the threat grid size: change the grid markup generation loop count (currently 200 cells) and adjust `.threat-grid` CSS (grid-template-columns / rows) for desired density.
- Adding new stat items: add an element with `class="stat-num"` and a `data-target` attribute (and optional `data-suffix`/`data-prefix`) so the stat counter code picks it up automatically.

## Integrations & external dependencies

- Google Fonts: fonts are loaded from fonts.googleapis.com — keep font-family references in CSS when changing typography.
- Formspree: modal signup posts to `https://formspree.io/f/mlgaryad` (hard-coded). If you change the form backend, update the fetch URL and payload shape.
- No other external APIs are used — the live demo behaviour is entirely simulated in-browser.

## Tests, linters, and build

- None present. There is no bundler, package.json, or test runner. Suggested minimal steps before larger changes:
  - Serve locally (see above) and smoke-test interactions (cursor, modal, log feed, threat grid, calculator).
  - Consider adding a tiny dev toolchain (prettier, eslint, or splitting CSS/JS files) only if you plan to scale the codebase.

## When writing or refactoring code

- Keep DOM ids stable or update both HTML and JS together.
- Preserve CSS variables in `:root` — use them for all colors/accents to keep theme coherence.
- Prefer non-destructive edits: add new helper functions rather than editing core loops in-place (e.g., add `addLogMessage()` wrapper rather than modifying `addLog()` directly).
- For accessibility improvements, note that interactive elements currently rely on visuals heavily; add keyboard handlers when introducing new modals or interactions.

## Where to look first for changes

- `main.html` — header comment: bottom script block (JS) and top <style> block (CSS) are the two primary areas you will edit.

---
If any section is unclear or you'd like more detail (for example, a mapping of all element ids to their JS usages with line references), tell me which area and I will expand or iterate.