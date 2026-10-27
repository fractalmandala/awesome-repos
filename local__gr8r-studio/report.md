---
slug: local__gr8r-studio
name: Gr8r Studio
github: unknown
license: unknown
scanned: 2026-10-27
verdict: ok
stack: [javascript, css, vite, spa, localstorage]
---

# Gr8r Studio

> Polished dependency-free PM SPA: coherent layers, hand-rolled focus, no tests or license.

**Verdict: ok** — genuinely hand-crafted and it builds/lints clean, but there is no LICENSE, no test of any kind, and a single 2,293-line action module carries every mutation.

## What it is
A project-management workspace UI: home dashboard, inbox, my tasks, projects with overview / board / list / table / calendar / timeline / files tabs, members, teams, activity, settings, auth and onboarding. It ships as a static Vite SPA with no backend: all data comes from `src/data/seed.js`, lives in one mutable global object, and is persisted to `localStorage` under `gr8r.v2`. Written in plain JavaScript ES modules that return HTML strings; no UI framework and no runtime dependency (`package.json` has devDependencies only).

## Facts
| | |
|---|---|
| GitHub | unknown — no git remote, no `repository`/`homepage` in `package.json`, no repo link in README |
| License | unknown — no `LICENSE`/`COPYING` file and no `license` field in `package.json` (`"private": true`) |
| Stack | Plain ES modules + CSS; Vite 6.4.4; ESLint 9.39.5 (flat config) + Prettier 3.9.9; `lucide` 0.460.0 used only by the build-time icon-subset plugin; Node ≥18 (`.nvmrc` = 22) |
| Size | 70 JS/CSS files, 13,853 LOC (8,698 JS in 52 files + 5,155 CSS in 18 files), plus `index.html` 24, `vite.config.js` 70, `eslint.config.js` 17. Lockfile and `node_modules` excluded |
| Activity | no git history in download (plain folder, no `.git`) |
| Tests | 0 test files, 0 assertions, no `test` script; CI exists but runs only lint, format check and build |
| Entry points | `index.html` → `src/main.js`; router/render loop `src/shell/render.js`; root layout `src/shell/layout.js`; build plugin `vite.config.js` |

Largest files: `src/actions/actions.js` 2,293 · `src/styles/overlays.css` 812 · `src/styles/components.css` 736 · `src/data/seed.js` 544 · `src/styles/shell.css` 439 · `src/pages/settings.js` 435 · `src/overlays/popovers.js` 398 · `src/styles/responsive.css` 355 · `src/styles/views/projects-files.css` 316 · `src/styles/motion.css` 314.

## Architecture
- `index.html` holds three hosts (`#app`, `#layer`, `#toasts`); `src/main.js` imports ~50 modules in dependency order, reads `location.hash` once, then calls `render()`.
- `src/shell/render.js` owns the loop: `render()` replaces `#app` and `#layer` innerHTML, restores focus/caret/scroll, and `go(route, params)` is the router (in-memory history, no URL writes). `src/shell/layout.js` builds the sidebar/topbar/content/bottom-nav HTML and maps 21 routes to page functions.
- Pages (`src/pages/`) and views (`src/views/`) are pure string builders from the store. `src/components/task-list.js` holds the shared row/cell renderers; `src/shell/view-engine.js` holds the filter/sort/group engine and the toolbar/chips used by four list surfaces.
- State is one mutable singleton in `src/core/store.js` (`S.data`, `S.prefs`, `S.views`, and 43 `S.ui` flags) with derived lookups (`allTasks`, `progressOf`, `canSee`) and a 150 ms debounced `save()`. `src/data/seed.js` is the demo workspace.
- Interaction is delegated: templates emit `data-a` / `data-in` / `data-key-*`, `src/actions/events.js` dispatches to the `A`/`IN`/`KEY`/`DBL`/`BLUR` tables in `src/actions/actions.js` (182 `A` handlers, 422 `data-a` sites). `mutate()` in `src/ui/toast.js` is the write choke point: offline guard → mutation → save → render.
- `src/overlays/drawer.js` composes the whole overlay layer: task drawer, modal stack, command palette, popovers and context menus; `src/overlays/palette.js` adds 17 commands plus scoped search.
- Boundaries are browser-only: `localStorage` (the only persistence), `navigator.clipboard`, a simulated upload pipeline (`handleFiles` ticks fake progress), Google Fonts loaded non-blocking, and Lucide icons inlined by a custom Vite plugin.
- No backend, no client URL routing, no test harness, no state management library.

Maps: [overview](scan-overview.html) · [actions module](scan-actions.html) · [shell module](scan-shell.html).

## Quality evidence
- Builds / type-checks: ✅ `npm run lint` exit 0; `npm run format:check` reports “All matched files use Prettier code style!”; `vite build --outDir /tmp/gr8r-build` succeeds — 56 modules → 358.11 kB JS (103.08 kB gzip) + 75.21 kB CSS.
- Tests: ❌ none. No `*.test.*`, no `__tests__`, no test runner in `package.json`, and `.github/workflows/ci.yml` has no test step.
- Structure: ⚠️ layers documented in README match the folders, but `src/actions/actions.js` is 2,293 lines / 26% of the JS and holds tasks, projects, sharing, members, notifications, settings, auth and prototype controls; `src/data/seed.js` is 544 lines of inline data; `src/overlays/` is 1,159 lines across 5 files.
- Error handling: ⚠️ 10 `try`/`catch` blocks, each with a short “safe to skip” comment; one `console.error` in the route catch (`shell/layout.js:69`); 0 `TODO`/`FIXME`/`HACK`; 205 `esc()` calls, but `actions.js:928/937` store contenteditable `innerHTML` into `t.desc` and `drawer.js:91` re-injects it unescaped.
- Docs: ⚠️ README is detailed and accurate about structure, layering, `data-a` convention, select wrapper, icon subset and CI; it does not say the app is front-end-only with fake auth/uploads, and comments still name an earlier “QUIRE” project (`core/utils.js`) and `src/00-icons.js` (`core/icons.js`).
- Dependencies: ✅ 6 devDependencies, 0 runtime dependencies, caret ranges plus `package-lock.json`; `lucide` is build-time only via `virtual:icons`.
- Maintenance: ⚠️ no git history in the download, so commit cadence is unknown; 0 TODO markers and a `README` screenshot (`docs/preview.png`, 540 KB) suggest a finished hand-off rather than active work.

## Slop signals
- [ ] README promises features that have no code — every advertised surface exists (`pages/*`, `views/*`, `settings.js`, `members.js`, `archive.js`).
- [ ] Large generated-looking files — `actions.js` is long but dense and hand-consistent (182 named handlers, domain comments), not repetitive boilerplate.
- [ ] Dead code / placeholder impls — 0 `todo!`/`unimplemented`/`TODO`; auth and upload are simulated but the UI says “prototype” in place (`pages/auth.js:60`, `settings.js:372`).
- [ ] Inconsistent architecture — one approach end to end: string render → global store → action table.
- [ ] One or two giant commits — cannot check: no `.git` in the download.
- [ ] Copied upstream without attribution or with the license removed — cannot verify; no license file exists either way.
- [ ] Doesn't build / obvious compile errors — it builds, lints and formats clean.
- [ ] Tests that assert nothing — no tests at all, so this signal does not apply.

## Verdict rubric
**ok** — real, coherent work that builds and reads well, with clear weak areas: no license, zero tests, a god action module, no URL routing and an unsanitised HTML sink. One rubric condition for `good` (license present) fails; 0 slop signals are ticked, so it is not `slop`.

## Worth stealing
- Focus/caret/scroll survival across a full `innerHTML` rebuild: `focusKey`/`FKEYS`/`tryFocus` and `[data-keep]` capture in `src/shell/render.js`, with opener restoration for every overlay.
- `mutate()` as a single write choke point (`src/ui/toast.js`): offline guard, save, render in one place, plus `snapshot()`/`restore()` for toast Undo.
- The Lucide subset Vite plugin (`vite.config.js`): scans source for quoted icon names and exports inline SVG through `virtual:icons`, so no icon runtime ships.
- Delegated `data-a` actions with one handler table (`src/actions/events.js`) — a framework-free way to keep markup declarative.
- Accessibility follow-through: `makeFocusable()` for non-native controls, `Tab` trapping in the keyboard layer, `aria-live` route announcements, skip link and focus return per overlay.

## Structural findings
- **actions.js is 2,293 lines and owns everything** — 182 `A` handlers, 36 `IN`, 7 `KEY` plus forms, auth, settings and prototype controls in one file: 26% of app JS. Domain splits (tasks, projects, account) are absent. — `src/actions/actions.js`
- **Every keystroke rebuilds the whole app** — `render()` replaces `#app.innerHTML` and `#layer.innerHTML`; `IN.palQ`, `IN.viewQ`, `IN.searchQ` and `IN.draft` call it on input, with focus/caret/scroll restored by hand across 176 `render()` sites. — `src/shell/render.js`
- **Task descriptions are stored and injected raw** — `actions.js:928/937` assign contenteditable `innerHTML` to `t.desc`; `drawer.js:91` interpolates it unescaped, so pasted markup runs on the next render. Everything else goes through `esc()`. — `src/actions/actions.js`

Also: no URL sync — `location.hash` is read once in `main.js:70` and never written, so browser Back exits the app; and 45 of the 52 JS files import the same global store.

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| actions | [scan-actions.json](scan-actions.json) | [scan-actions.html](scan-actions.html) |
| shell | [scan-shell.json](scan-shell.json) | [scan-shell.html](scan-shell.html) |
