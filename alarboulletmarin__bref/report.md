---
slug: alarboulletmarin__bref
name: Bref
github: https://github.com/alarboulletmarin/bref
license: MIT
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, desktop, markdown]
---

# Bref

> A fast, minimal Markdown note app in Rust/GPUI: plain files, live styling, diagrams, CSV, git sync.

**Verdict: good** — 49k lines that build offline and pass 294 tests, with a layer rule enforced by a test, an explicit incremental-layout invariant, and no `todo!()` anywhere in `src/`.

## What it is
Bref is a desktop note-taking app for a folder of plain `.md` files (the "vault"). It renders Markdown live while keeping the buffer raw, and adds own-file formats next to it: CSV/TSV grids, hand-drawn diagrams stored as SVG with their source in `<metadata>`, Kanban boards written as Markdown tasks, and `![[Note#Heading]]` embeds. It is a single Rust binary on GPUI 0.2.2, shipped for Linux (pacman, make), macOS (dmg) and Windows; one crate, no workspace.

## Facts
| | |
|---|---|
| GitHub | https://github.com/alarboulletmarin/bref (from `Cargo.toml` `repository`; the download has no `.git`) |
| License | MIT (`LICENSE`, also `license = "MIT"` in `Cargo.toml`) |
| Stack | Rust 2024, rust-version 1.88; `gpui` 0.2.2 (crates.io, not git), `notify` 7, `mermaid-rs-renderer` 0.3.1, `latex-rust` 2.1.0, `resvg`/`usvg` 0.45.1, `regex`, `serde_json`, `sha2`, `unicode-segmentation`; IBM Plex fonts vendored in `assets/fonts` |
| Size | 139 `.rs` files, 49,377 LOC total; 108 non-test files / 41,585 LOC. Largest: `logic/markdown/mod.rs` 1,705 · `logic/diagram/mod.rs` 1,692 · `views/editor/layout.rs` 1,589 · `platform/vault.rs` 1,321 · `logic/table.rs` 1,295 · `views/palette.rs` 1,196 |
| Activity | no git history in download (folder `bref-main`); version 0.4.2 |
| Tests | 294 passed / 10 ignored (`cargo test --offline`, 162 s); 130 inline `#[cfg(test)]` modules; 7,792 LOC of end-to-end UI tests in `src/shell/tests`; CI matrix ubuntu/macos/windows (`.github/workflows/ci.yml`) |
| Entry points | `src/main.rs` → `shell::run()` (`src/shell/mod.rs:298`); `bref --capture` opens `src/shell/capture.rs` |

## Architecture
A strict four-layer stack, documented in `ARCHITECTURE.md` and enforced by a test:

- `src/logic/` (13.2k LOC) — pure functions: Markdown analysis, CSV tables, cell formulas, embeds, diagrams, prefs, history. No gpui, no disk.
- `src/platform/` (2.5k) — the outside world: `vault.rs` (scan, atomic write, watch, trash, backup), `git.rs` (spawns `git`), `update.rs` (one `curl` a day), `clock.rs`.
- `src/ui/` (2.1k) — theme tables (13 themes), fonts/icons, widgets, `figure.rs` (Mermaid/LaTeX → pixels via resvg).
- `src/views/` (16.5k) — gpui entities: `editor`, `nav`, `palette`, `sheet`, `canvas`, `graph`, `viewer`, `kanban`.
- `src/shell/` (7.1k) — `Shell`, the window: owns panes and views, consumes their events with `cx.subscribe_in`, dispatches `actions!` from `src/actions.rs` through the keymap.
- `src/shell/tests/` — the whole app driven by simulated keystrokes, one chapter per feature (`end_to_end/`, 21 files).

Maps: [overview](scan-overview.html) · [shell](scan-shell.html) · [editor](scan-editor.html) · [logic](scan-logic.html).

## Quality evidence
- Builds / type-checks: ✅ `cargo check --offline --all-targets` finished in 46 s with 2 unused-import warnings (`platform/mod.rs:8`, `shell/mod.rs:40`). No network needed: `Cargo.lock` is complete.
- Tests: ✅ `cargo test --offline` → `294 passed; 0 failed; 10 ignored` in 162 s. The ignored ones are benchmarks/fuzzers by design (`src/shell/tests/benches.rs`). Includes `layers_only_depend_downwards` and `layout_reuse_matches_full_layout`.
- Structure: ✅ 24 shell files, largest 956 lines, one subject each; 20 editor files, largest 1,589 (`layout.rs`). ⚠️ Two god-modules by size: `logic/markdown/mod.rs` (1,705, 29 public fns) and `logic/diagram/mod.rs` (1,692).
- Error handling: ✅ only 32 `unwrap()`/`expect(` outside `#[cfg(test)]` blocks (the raw grep is 571, mostly module-local tests); no `todo!()`/`unimplemented!()`; no `TODO`/`FIXME` in `src`. Known hot spot flagged in-code: synchronous disk write on the UI thread (`ponytail:` at `shell/notes.rs:318`).
- Docs: ✅ `README.md` (637 lines) and `ARCHITECTURE.md` (92) match the tree; per-file `//!` headers everywhere explain intent, not syntax. One drift: README says "a dozen themes", `ui/theme.rs` has 13.
- Dependencies: ✅ 18 direct deps, all pinned by `Cargo.lock`; the manifest comments explain each one (`Cargo.toml:1-40`), including why `notify` is 7 not 8. No exotic or abandoned crates visible.
- Maintenance: ✅ no dead code, no placeholder impls, consistent 0.4.2; activity cannot be judged from a zip without `.git`.

## Slop signals
- [ ] README promises features that have no code
- [ ] Large generated-looking files
- [ ] Dead code / placeholder impls
- [ ] Inconsistent architecture
- [ ] One or two giant commits containing the whole project
- [ ] Copied upstream without attribution
- [ ] Doesn't build / obvious compile errors
- [ ] Tests that assert nothing

None ticked. The 30+ README feature claims checked against code all have an implementation (e.g. CriticMarkup comments → `logic/markdown/mod.rs:800`, `logic/embed.rs` → `views/editor/embed.rs`, CSV → `logic/table.rs`, git conflict → two notes in `platform/git.rs:119`).

## Verdict rubric
- **good**: builds, clear structure, real tests, license present, ≤ 1 slop signal. ✅

## Worth stealing
- `src/shell/tests/mod.rs:25` — `layers_only_depend_downwards`: a test that greps each layer for banned imports (after stripping comments), so the architecture diagram cannot rot.
- `src/shell/tests/layout.rs` — `layout_reuse_matches_full_layout`: compares the incremental layout path against a full one, making the perf fast-path verifiable.
- `src/shell/notes.rs:396` — debounced autosave with a `save_gen` counter, so a burst of keystrokes writes once and a stale timer is ignored.
- `src/logic/diagram/file.rs` — a file format that is a plain SVG carrying its own source in `<metadata>`, plus `Diagram::needed` to write the lowest compatible version.
- `src/logic/table.rs` — index-only CSV parsing (`Arc<String>` + 12-byte record bounds), the kind of decision that makes "a million rows" plausible.

## Structural findings
- **Shell is one struct, split by subject** — `struct Shell` (`src/shell/mod.rs:77`) carries 20+ fields; 23 `impl Shell` files each own one subject, so no shell file exceeds 956 lines, but every feature reaches the same struct. (`src/shell/mod.rs`)
- **markdown/mod.rs is a 1.7k-line toolbox** — 29 public functions in one file (classify, emphasis, links, comments, headings, index, relink) that most views import directly. (`src/logic/markdown/mod.rs`)
- **The editor is a 766-line, 83-field struct** — text, cursors, undo, find, bar, figure cache, embeds and hitboxes all live on `Editor`; the 20 part files are `impl Editor` on the same struct. (`src/views/editor/mod.rs`)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| shell | [scan-shell.json](scan-shell.json) | [scan-shell.html](scan-shell.html) |
| editor | [scan-editor.json](scan-editor.json) | [scan-editor.html](scan-editor.html) |
| logic | [scan-logic.json](scan-logic.json) | [scan-logic.html](scan-logic.html) |
