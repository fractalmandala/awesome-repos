---
slug: henrikogaard__rista
name: rista
github: https://github.com/henrikogaard/rista
license: MIT
scanned: 2026-10-27
verdict: good
stack: [rust, gpui-kit, markdown, acp, mermaid, katex, pty]
---

# rista

> Native Rust Markdown workspace: folders as pages, bases, graph, reviewable agents.

**Verdict: good** — 381 inline `#[test]` functions, zero todo!/TODO/FIXME, real docs (DESIGN tokens + ARCHITECTURE.md), and the reviewable-ACP feature is fully implemented; the price is a 15.6k-line app.rs.

## What it is
Rísta is a native macOS/Linux Markdown workspace written in Rust on gpui-kit 0.7.1 (the author's fork of an Obsidian-like kit; a `gpui-base` patch commit is pinned). Folders open as living pages; notes are plain `.md` with YAML frontmatter; `.base` files define Obsidian-compatible databases (filters/formulas/6 view types); a graph shows note links; an ACP agent proposes writes as accept/reject diffs. Ships as a Developer-ID signed macOS app with Sparkle updates, a bundled CLI (`src/cli.rs` 2,200 LOC with headless read-only runs and socket-coordinated writes), and a Mermaid/KaTeX offline diagram stack.

## Facts
| | |
|---|---|
| GitHub | https://github.com/henrikogaard/rista (README release link; gpui fork same owner) |
| License | MIT (LICENSE + Cargo.toml `license = "MIT"`; © 2026 Henrik Øgård) |
| Stack | Rust 2021, gpui-kit 0.7.1 (+ patched gpui-base rev b470bda), markdown 1.0, notify 7, ureq 3, portable-pty/vt100, ratex (LaTeX), merman+resvg 0.45, KaTeX fonts vendored under third_party/ (OFL) |
| Size | 65 rs files in `src/`, 92,217 LOC total |
| Activity | no git history in download; CHANGELOG v0.2.0 present |
| Tests | 381 `#[test]` functions inline across 48 src files; release CI (`.github/workflows/release.yml`) only in this drop |
| Entry points | `src/main.rs` (gui/headless/CLI), `gpui_kit::open_window` in `src/windows.rs`, `src/cli.rs` for `rista` on the shell |

## Architecture
- **src/main.rs** — three startup modes: Help/Version, Headless (read-only CLI or socket-coordinated writes), GUI with url/IPC listeners and a style-snippet file watcher.
- **src/windows.rs + src/app.rs** — WindowManager global and a 15,651-line Workspace entity that holds tab/pane layout, ~80 on_action handlers, palette, quick open, notification strips and more.
- **src/vault.rs** — opened folder: tree state, flattened note index, notify-watcher with burst debounce feeding `VaultEvent`s.
- **src/document.rs** — one open note; editor state, autosave, disk sync conflict path, spellcheck, vim mode. Created per file, subscribed by every surface that edits.
- **src/bases.rs** — frontmatter database: an expression language (Lit/Parser), formulas, rollups (`sum/avg/list` over relations), and `BaseView` rendering `table|cards|gallery|kanban|board|calendar`.
- **src/agent.rs + agent_panel.rs** — ACP adapter subprocess over mpsc/1MB frames; the panel shows plan/tool calls, permission prompts, and before/after diffs of writes.
- **src/graph.rs + graph_layout.rs** — vault graph/local graph with a shared force-repulsion + fit + label-grid layout helper.
- **src/search*.rs + preview.rs** — search expression engine (2.3k parser) over the vault; preview pipeline renders Markdown (preview/split/source tabs) with emoji, math (ratex) and Mermaid SVG rasterized offline.
- **src/cli.rs + cli_ipc.rs** — headless CLI against the socket; cli_ipc keeps the instance lock and 4-byte-framed request/response protocol (1MB cap).

## Quality evidence
- Builds / type-checks: not run (cargo fetch/network extra would be needed; not attempted).
- Tests: ✅ 381 `#[test]` fns in 48 src files — no external test dir, everything co-located (`bases.rs`, `search_query.rs`, `agent.rs`, `terminal_session.rs`, …).
- Structure: ⚠️ flat 65-file `src/` works until app.rs; `src/app.rs` alone is 15,651 LOC and grows with each feature (biggest single file by 2× vs `bases.rs` 8,692).
- Error handling: ✅ 0 todo!/unimplemented! and 0 TODO/FIXME/HACK matches across src; 1,034 `unwrap()/expect(` matches (mostly context/window accessors appropriate behind asserts in a GUI).
- Docs: ✅ README claims match code (folders-as-pages ↔ `folder.rs`/`folder_dashboard.rs`, bases ↔ `bases.rs`, reviewable agent ↔ `agent_panel.rs`, offline diagrams ↔ `Cargo.toml`); DESIGN.md ships token.json/ts/css + a Python exporter; `docs/ARCHITECTURE.md` and `TEST-CASES.md` exist.
- Dependencies: ~35 deps; pinned pins look pristine (`ratex-parser = "=0.1.14"`, `merman = "=0.8.0"`); gpui-base patch pinned to a rev in the author's fork with an explanatory comment.
- Maintenance: not checked (no git history in download); CHANGELOG + README's alpha notes and Release checklist present.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — commit cadence unverifiable in this zip
- [ ] Copied upstream without attribution — *not fired*
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, thorough docs, honest scope discipline, 1 slop signal (no history in download).

## Worth stealing
- **Reviewable agent writes**: `PendingWrite` before/after with explicit accept/reject before any file lands (`src/agent_panel.rs`).
- **One query engine, two consumers**: `src/bases.rs` DSL powers both GUI views and headless CLI; `src/search_query.rs` the same for search.
- **Single-tee window startup**: `main.rs` cleanly separates Help/Version, Headless (read-only CLI vs socket writes) and GUI with explicit on_action defers.
- **Document-event fanout**: `BaseView` recomputes from `DocumentEvent::Changed/Saved` and bumps `doc_epoch` rather than polling.
- **Bundled CLI + socket**: `src/cli_ipc.rs` gives a persistent app a CLI contract with instance-lock + 4-byte framed protocol (1MB msg cap).

## Structural findings
1. **app.rs is a 15,651-LOC monolith** — Workspace entity holds tabs, panes, keyboard handling, palette, action wiring and graph/terminal/panel triggers in one file; splitting per panel is the top lever. (alert)
2. **flat module layout** — 65 top-level `src/*.rs` files with no sub-module grouping; conventions exist (pdf_* trio, note_composer*) but discovery cost grows slowly. (info)
3. **1034 unwrap/expect** — GUI-context accessors dominate; mostly context/window accessors; an audit pass around the `vault.rs` watcher would be the next step. (warn)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| bases | [scan-bases.json](scan-bases.json) | [scan-bases.html](scan-bases.html) |
| agent | [scan-agent.json](scan-agent.json) | [scan-agent.html](scan-agent.html) |
