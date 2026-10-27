---
slug: schmayterling__hibi
name: hibi
github: https://github.com/schmayterling/hibi
license: GPL-3.0-only
scanned: 2026-10-27
verdict: good
stack: [typescript, electron, react, tiptap, codemirror, markdown, node]
---

# hibi

> Electron note editor: rich and source views over one document engine, with 42 bundled addons.

**Verdict: good** — a large, deliberately layered app (main / preload / renderer / shared / addons) with a real text engine, typechecks clean, 346 test files, and one guarded gate in front of 130 IPC channels.

## What it is
hibi is a desktop editor for local notes and documents: Markdown, LaTeX, and other formats, with a formatted (TipTap) view, a source (CodeMirror) view, or both side by side. Files stay on disk in a folder ("workspace") it can open, index and watch, and it aims at Obsidian-vault compatibility on a best-effort basis. It ships as an Electron app for macOS, Windows and Linux (AppImage, deb, flatpak) with nightly releases.

## Facts
| | |
|---|---|
| GitHub | https://github.com/schmayterling/hibi (from `package.json` `homepage` and every badge/link; the download has no `.git`) |
| License | GPL-3.0-only (`LICENSE.md`, and `license` in `package.json`) |
| Stack | TypeScript 7.0.2, Electron 45, electron-vite 6, React 19.3, TipTap 3.32, CodeMirror 6, Vite 7.3.7, Biome 2.5.15, Vitest 4.1.12, Playwright 1.64, Node ≥ 22.18 (`.nvmrc` 24); marked/markdoc/MDX, mermaid 12, KaTeX 0.18, typst-ts, MiniSearch, DOMPurify, d3-force, unzipper |
| Size | 607 `.ts`/`.tsx` files, 102,730 LOC in `src/` (+ 6,621 CSS); 346 test files, 105,327 LOC in `tests/`; 27 internal docs pages, 340 Markdown files in the docs snapshot; 42 addons; 66 scripts |
| Activity | no git history in download (folder `hibi-main`); version 0.2.0, nightlies from `main` |
| Tests | `node --test` suites (346 files, sharded 4× across 3 OS in `.github/workflows/check.yml`); Electron-driven cases through Playwright's `_electron` (`tests/electron.mjs`); Vitest + CodSpeed benches; desktop benchmarks in `bench/desktop` |
| Entry points | `src/main/index.ts`, `src/preload/index.ts`, `src/renderer/src/main.tsx`, `src/site/main.tsx` (docs viewer), addon entries under `src/addons/*/index.tsx` |

## Architecture
- **Process split**: `src/main` owns windows, menu, validated IPC, disk and native addons; `src/preload` exposes only named calls; `src/renderer` is a sandboxed React app with no Node access.
- **One document engine**: `src/shared/source-buffer.ts` is a persistent chunk tree with immutable snapshots; `document-session.ts` adds history groups and projections; both editors read it.
- **Two views, one buffer**: TipTap (`Editor.tsx`) and CodeMirror (`SourceEditor.tsx`) exchange `SourceOperation`s through `plain-source-sync.ts` and `rich-source-session.ts`.
- **Out-of-thread analysis**: `document-worker-service.ts` keeps its own store for find, metadata and Markdown semantics; `document-worker-client.ts` restarts it on stalls.
- **Addons are the feature layer**: `src/addons/api.ts` (API v2) declares capabilities; 42 bundled addons contribute flavors, commands, panels, settings, and whole windows. Main validates packages (`addon-inventory.ts`) and signatures (`addon-trust.ts`).
- **Docs site is a separate consumer**: `src/site/main.tsx` renders the same content without the Electron bridge.

Maps: [overview](scan-overview.html) · [main](scan-main.html) · [renderer](scan-renderer.html) · [addons](scan-addons.html) · [shared](scan-shared.html).

## Quality evidence
- Builds / type-checks: ✅ `tsc --noEmit -p tsconfig.node.json` and `-p tsconfig.web.json` both exit 0 with the repo's own TypeScript 7.0.2 (run here, no install). The Electron build itself was not run.
- Tests: ✅ spot-ran `node --test` on `markdown-source-model`, `source-buffer` and `atomic-writes` (67 pass) plus `security` and `hotkeys` (4 pass). The full 346-file suite, including the Electron/Playwright cases, was not run (it is sharded in CI).
- Structure: ⚠️ clear boundaries (7 top-level dirs, contracts in `src/shared`), but three files carry the shell: `renderer/src/Editor.tsx` 3,367, `renderer/src/main.tsx` 3,315, `renderer/src/addons.ts` 2,648, and `main/index.ts` 2,439.
- Error handling: ✅ 2 `TODO`/`FIXME` hits, both the string `'TODO: '` in `writing-suggestions`; 1 `@ts-expect-error` for Svelte's server runtime; 2 `any` in a single user addon; 56 `biome-ignore` comments, each with a written reason (e.g. `ui/Sidebar.tsx:164`).
- Docs: ✅ `docs/ai-agents/` (27 pages) documents architecture, validation, performance, releases and addon APIs; `AGENTS.md` is short and points there. `docs/` itself is a generated snapshot (ignored in `.gitignore`, pinned by `docs-revision`).
- Dependencies: ⚠️ 90 direct deps (83 dev, 7 runtime) pinned exactly; Electron and the CodeMirror/TipTap stack are heavy but current. `overrides` pins `lodash-es`, and weekly grouped updates come from `.github/dependabot.yml`.
- Maintenance: ✅ no dead code or placeholder impls seen; `npm run check` chains lint, docs, copy and tests; nightly + release workflows; activity cannot be judged without `.git`.

**Lint note**: `biome check src` reports 0 errors and 341 warnings on this download. `biome check .` reports 24 errors, all in the generated `.hibi-addon-build/` directory (a build artifact of `scripts/addon-packages.ts`, not covered by `.gitignore`), so `npm run lint` on this exact folder fails on generated files rather than on source.

## Slop signals
- [ ] README promises features that have no code
- [ ] Large generated-looking files
- [ ] Dead code / placeholder impls
- [ ] Inconsistent architecture
- [ ] One or two giant commits containing the whole project
- [ ] Copied upstream without attribution
- [ ] Doesn't build / obvious compile errors
- [ ] Tests that assert nothing

None ticked. The one visible borrowing is credited in place: `src/addons/keybeats/LICENSE.kbsim.md` and `LICENSE.keybeats.md` sit beside the engine, and `@replit/codemirror-vim` is a normal dependency rather than copied code.

## Verdict rubric
- **good**: typechecks clean, clear structure, heavy tests, license present, ≤ 1 slop signal. ✅

## Worth stealing
- `src/main/index.ts:1340` — a single `handle()` helper that wraps every IPC registration with the same sender/frame/URL check; 130 channels cannot skip it.
- `src/shared/source-buffer.ts` — a chunk tree with subtree sums, immutable snapshots and owner ranges: one model for editing, search, folding and crash recovery.
- `src/shared/document-journal.ts` — checkpoint + replay verified before use, so a crash does not silently produce a wrong file.
- `src/addons/api.ts` + `src/main/addon-trust.ts` — capability-gated addons with a separate, auditable signature/revocation step.
- `src/shared/local-diagnostics.ts` — a redaction policy and byte/frame budgets in the contract itself, so diagnostics cannot carry document text.

## Structural findings
- **Two 3k-line files hold the app shell** — `src/renderer/src/main.tsx` (3,315, one `App` component) and `Editor.tsx` (3,367); only four views are lazy-split.
- **One guarded gate for 130 IPC channels** — every registration in `src/main/index.ts` goes through `handle()` → `trustedWindow`, which is also where benchmark timing is injected.
- **Rich and source views share one model** — `SourceStore` in `src/shared/source-buffer.ts` is edited by both `Editor.tsx` and `SourceEditor.tsx` through `plain-source-sync.ts`.

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| main | [scan-main.json](scan-main.json) | [scan-main.html](scan-main.html) |
| renderer | [scan-renderer.json](scan-renderer.json) | [scan-renderer.html](scan-renderer.html) |
| addons | [scan-addons.json](scan-addons.json) | [scan-addons.html](scan-addons.html) |
| shared | [scan-shared.json](scan-shared.json) | [scan-shared.html](scan-shared.html) |
