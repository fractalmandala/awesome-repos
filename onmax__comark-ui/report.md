---
slug: onmax__comark-ui
name: Comark UI
github: https://github.com/onmax/comark-ui
license: MIT
scanned: 2026-10-27
verdict: good
stack: [typescript, pnpm, comark, markdown, html, vue, react, svelte]
---

# Comark UI

> 45-package pnpm workspace: Markdown components as parse plugins plus HTML, Vue, React and Svelte renderers.

**Verdict: good** — one package shape repeated 45 times, the registrations enforced by tests, parse-time work so no renderer ships Vega or Temml, and zero TODO/FIXME in shipped code.

## What it is
Comark UI is the component library for [Comark](https://comark.dev): a set of Markdown blocks (tabs, cards, charts, annotated screenshots, code changes, diagrams, quizzes…) that render as static, script-free-where-possible HTML, with Vue, React and Svelte equivalents. It is a pnpm monorepo of 45 published npm packages, all at one version (`0.6.0`), plus a docs site, an agent skill and a linter. Consumers install one umbrella package (`comark-ui`) or individual `comark-*` packages.

## Facts
| | |
|---|---|
| GitHub | https://github.com/onmax/comark-ui (from `package.json` `repository`; the download has no `.git`) |
| License | MIT (`LICENSE`, and `license: MIT` in every package manifest) |
| Stack | TypeScript 6, pnpm 11.18.0, Node 24 (>=22.12), `comark` 0.8.1, `@comark/html|vue|react|svelte` 0.8.1, Vite+ 1.1.0 / tsdown 0.23, Vue 3.5, React 19, Svelte 5.57, TipTap 3.31, Vega/Vega-Lite + flint-chart 0.5, d3-sankey, KaTeX/Temml, Playwright 1.64 |
| Size | 45 packages; 428 files / 33,782 LOC under `packages/*/src`; 93 test files / 13,886 LOC; 19 Playwright specs / 3,135 LOC; docs 4,063 LOC TS over 357 Markdown pages; 46 skill files |
| Activity | no git history in download (folder `comark-ui-main`); one version 0.6.0 for every package |
| Tests | 93 `*.test.ts` (unit + HTML snapshot-style), `test/workspace.test.ts` invariants, 4 root tests, 19 e2e specs; CI has `check` (format/lint/typecheck/test/doctor/docs) and `e2e` (Chromium) jobs on Node 24 and 22.18 |
| Entry points | `packages/ui/src/index.ts` (`preset()`), `.../html.ts`, `.../vue.ts`, `.../react.ts`, `.../svelte/index.js`, `.../editor.ts`; `lint-comark` CLI in `packages/lint/src/cli.ts` |

## Architecture
- **One package shape**: `src/index.ts` is the parse plugin, `src/{html,vue,react,svelte}` the renderers, optional `src/client.ts`/`script.ts`/`styles.ts`/`read.ts`. `defineComponentsPackage()` in `tsdown.shared.ts` requires exactly those entries and copies `src/svelte` as source.
- **Parse-time over render-time**: charts draw their SVG while parsing (`packages/charts/src/index.ts`), math typesets to MathML (`packages/math`), diagrams call Kroki (`packages/diagrams`), icons resolve to data URIs (`packages/icon`). Renderers only print attributes.
- **One reader, two consumers**: annotate, charts and quiz each keep `read.ts` shared by the plugin and the renderer, so validation and drawing cannot disagree.
- **`comark-ui` is the ordering layer**: `preset()` (index.ts:177) registers 40 plugins in the one order the docs rely on and shares a single `createIcons()`.
- **The linter is a peer**: `comark-linter` reads `meta.issues` from the same plugins and catalogs 285 rules, exposed as a library, CLI, ESLint plugin and Vite Doctor extension.

Maps: [overview](scan-overview.html) · [ui](scan-ui.html) · [lint](scan-lint.html) · [annotate](scan-annotate.html) · [charts](scan-charts.html).

## Quality evidence
- Builds / type-checks: not run — there is no `node_modules` in the download and `pnpm install` needs the network. CI runs `pnpm build`, `pnpm check` and `pnpm test` on every PR (`.github/workflows/ci.yml`).
- Tests: ✅ 93 test files (13,886 LOC) beside the packages, 4 root invariant tests, 19 Chromium specs (3,135 LOC). `test/workspace.test.ts` asserts manifest invariants; `test/doctor.test.ts` asserts every lint rule has a docs page that behaves.
- Structure: ✅ 45 packages, each under ~1.4k LOC except `charts/src/draw.ts` (1,369), `charts/src/read.ts` (1,065) and `lint/src/index.ts` (939). One shared build preset; no package invents its own layout.
- Error handling: ✅ zero `TODO`/`FIXME`/`HACK`; only 4 `oxlint-disable` comments, each with a written reason (`packages/ui/src/index.ts:110`, `react.ts:46`, `lint/src/index.ts:50`); `lint()` documents "It never throws"; 76 `catch` blocks around optional imports and network fetches.
- Docs: ✅ `README.md` (637 lines) lists one row per package (43 rows plus the umbrella) and matches the tree; 357 Markdown pages plus 48 linter-rule pages; `AGENTS.md` walks every package and its reasoning. One drift to watch: README says "40+ blocks" while `packages/` holds 45 packages (two are the umbrella and the linter, not blocks).
- Dependencies: ✅ 50 pinned catalog entries in `pnpm-workspace.yaml`, one `pnpm-lock.yaml`, Renovate with automerge for dev/patch updates and pinned Action digests (`renovate.json`).
- Maintenance: ✅ `packageManager`, `.node-version`, provenance publishing (`publishConfig.provenance`), pkg-pr-new previews, a release workflow; activity cannot be judged from a zip without `.git`.

## Slop signals
- [ ] README promises features that have no code
- [ ] Large generated-looking files
- [ ] Dead code / placeholder impls
- [ ] Inconsistent architecture
- [ ] One or two giant commits containing the whole project
- [ ] Copied upstream without attribution
- [ ] Doesn't build / obvious compile errors
- [ ] Tests that assert nothing

None ticked. The one borrow is credited in place: `packages/annotate/src/marks.ts` names AnnotationKit in its header.

## Verdict rubric
- **good**: builds in CI, clear structure, real tests, license present, ≤ 1 slop signal. ✅

## Worth stealing
- `test/workspace.test.ts` — a test that fails when a new package forgets its manifest fields, skill reference or docs page. The checklist lives in code, not in a CONTRIBUTING paragraph.
- `tsdown.shared.ts` — two config factories (`definePackage`, `defineComponentsPackage`) that make 45 packages build identically, including the `style.css` emit and Svelte source copy.
- `packages/annotate/src/read.ts` — one validator/drawer shared by plugin and renderer, with `issueRules` mapping each message to a linter code.
- `packages/charts/src/index.ts` — drawing in `post()` with dynamic imports, so the client never downloads Vega.
- `packages/zoom/src/script.ts` — a self-contained browser script whose source the HTML renderer inlines, and which Vue/React/Svelte start on mount.

## Structural findings
- **Renderer entries are hand-maintained** — `packages/ui/src/html.ts` and `index.ts` list ~40 imports by hand; a new plugin means editing the entries, the option type, the preset order, the linter catalog and the skill, five places. (`packages/ui/src/index.ts`)
- **Two files carry most of the complexity** — `packages/charts/src/draw.ts` (1,369) plus `read.ts` (1,065) are half the charts package; `packages/lint/src/index.ts` is 939 lines. (`packages/charts/src/draw.ts`)
- **Positions in the linter are reconstructed** — Comark nodes have no positions, so block lines come from markdown-exit tokens and everything else is found by searching a blanked copy of the prose. (`packages/lint/src/source.ts`)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| ui | [scan-ui.json](scan-ui.json) | [scan-ui.html](scan-ui.html) |
| lint | [scan-lint.json](scan-lint.json) | [scan-lint.html](scan-lint.html) |
| annotate | [scan-annotate.json](scan-annotate.json) | [scan-annotate.html](scan-annotate.html) |
| charts | [scan-charts.json](scan-charts.json) | [scan-charts.html](scan-charts.html) |
