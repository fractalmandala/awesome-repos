---
slug: jeanc18rlos__fanta-edit
name: fanta-edit
github: unknown
license: GPL-3.0-or-later
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, skia, metal, kiwi, mcp, tauri, sqlite]
---

# fanta-edit

> Native macOS design editor on a Zed fork: .fig import, FNX-save canvas, agent hooks.

**Verdict: good** — 156 test files, hardened CI (check, fanta-lint, runtime, release, windows, skia-source-compat), one clean design-op funnel shared by humans and agents, and a disciplined FNX format under Git; the weak spot is a 23k-LOC canvas entity plus 44 uncommitted modified files in the drop.

## What it is
Fanta is "your design is source": a native macOS (Apple Silicon) design editor built as a fork of Zed. Import a Figma `.fig`, create a design, or open a Fanta project folder; canvas changes save as readable FNX text (`fanta.json`, `page.fnx`, `doc/variables.json`) beside the project so Git can diff them and agents can edit through the Code workspace or MCP. Distributed as a signed/notarized "Build45" public alpha (fantaisa.net); also buildable as a Tauri-ish web build (`Dockerfile-distros`, `fanta-site`). All the design-engine work lives in `crates/fanta-*` plus `crates/fig_viewer` (~470k LOC under those, fig_viewer 185k); the rest (~920k LOC) is the intact Zed platform (gpui fork now published from `squidred-dev/fanta-ui`, editor, project, agent/vim panels).

## Facts
| | |
|---|---|
| GitHub | unknown (README links: 9× `github.com/jeanc18rlos/fanta-edit`, 3× `github.com/squidred-dev/fanta-ui`; no git remote in the download) |
| License | GPL-3.0-or-later (README license section; `LICENSE-GPL` full GPL-3; `Cargo.toml` workspace `license = "GPL-3.0"`, per-crate GPL-3 duplicates; `LICENSE-APACHE` retains Zed's 2022-2025 headers for upstream files) |
| Stack | Rust (rust-toolchain pinned), forked gpui (`fanta-gpui` 0.5.0 from fanta-ui rev d4e410c), vendored skia-bindings 0.153.3 + Metal present, kiwi/fig import, context-server MCP, sqlez/sqlite, clap CLI |
| Size | 1,422 tracked .rs files; 1.39M LOC on disk. Top: fig_viewer 185k, editor 154k, project 97k, agent 90k, agent_ui 84k, workspace 51k, fanta-render 47k, vim 47k, git_ui 38k, fanta-doc 35k |
| Activity | git history: single squashed commit 2026-10-25 (author dev@fanta.local), plus 44 modified files in the working tree — downstream agent work, not upstream history |
| Tests | 156 `tests/`/`benches/`/`*_tests.rs` files tracked; `tests/` dirs in fanta-format, fanta-render, fanta-fig-interop, fanta-tools, agent, project, worktree...; CI: check.yml, fanta-lint.yml, fanta-runtime.yml, release-macos.yml, windows.yml, skia-source-compatibility.yml |
| Entry points | `crates/fanta/src/main.rs` (app), `crates/cli/src/main.rs`, `crates/fanta-engine/src/main.rs`, `crates/fanta-site/src/main.rs` |

## Architecture
- **crates/fanta** — app entry: crash handler, built-in themes, workspace bootstrap, registration of open/settings actions. [overview](scan-overview.html)
- **crates/workspace** (51k) — Zed workspace shell; registered `fig_viewer::init` after `editor::init` so `fanta.json` resolves to the canvas, not a JSON buffer (`crates/fig_viewer/src/workspace_hooks.rs`).
- **crates/fig_viewer** (185k) — the canvas: view.rs 23k-LOC entity (camera/tools/gestures, 40+ actions), document.rs 12k (live doc, debounced FNX writes), design/properties/motion/prototype panels, code_workspace 4.8k, draw_wand, generation_workspace, live_mcp 1.5k. [fig-viewer map](scan-fig-viewer.html)
- **crates/fanta-doc → fanta-render → fanta-present** — engine split: scene model (`FigDocument/Scene/Viewport`), Skia raster backend with one shared scene walk (headless readback or Metal-backed `render_to_canvas`), and Metal window presentation. [fanta-render map](scan-fanta-render.html)
- **crates/fanta-fig-interop / fanta-format** — `.fig` kiwi import into the scene; FNX parse/write of the Git-diffable project tree.
- **crates/design_surface + fig_viewer/agent_surface + live_mcp** — one `DesignOp` provider; built-in agent tools (`agent_ui`) and external agents (Claude Code, codex) over a Unix-socket MCP (`fanta_live_mcp.json` discovery) share it.
- **Zed core** — editor/project/agent_ui/remote/lsp kept intact; settings store, sqlez session persistence, command palette all inherited.

## Quality evidence
- Builds / type-checks: not run — full clean build of a 1.4M-LOC workspace exceeds an autonomous pass; CI runs `cargo check` + license check per crate (`.github/workflows/check.yml`).
- Tests: ✅ 156 test/bench files, in-module `tests/` dirs for the Fanta engine crates; `./script/smoke-mcp` exercises the agent bridge.
- Structure: ⚠️ engine crates are cleanly layered (fanta-doc / fanta-format / fanta-fig-interop / fanta-render with `ARCHITECTURE.md` §13 precision contract), but `fig_viewer/src/view.rs` is 23,229 LOC and inherits Zed's fat modules (editor 154k).
- Error handling: ✅ 0 `unwrap()` matches in fig_viewer/fanta-render/fanta-doc/src (anyhow/gpui Result style); 1 TODO/FIXME/HACK match across the Fanta crates; workspace-wide, inherited Zed code has 40 `todo!/unimplemented!` and 7,386 `unwrap|expect` (mostly upstream editor).
- Docs: ✅ README (7.2k) matches code — FNX saving ↔ `fanta-format`/`document.rs`, agent quickstart ↔ `live_mcp.rs`, limitations section distinguishes verified behavior explicitly; `docs/alpha/PUBLIC_ALPHA.md` documents signing/notarization steps. No claimed feature found without an implementation.
- Dependencies: 149-crate workspace, `grep renovate.json` + `Cargo.lock` (416k) pinned; gpui pulled from a sibling repo at a pinned rev.
- Maintenance: git history is one squash commit (2026-10-25) + 44 dirty working files — upstream upstream cadence is not visible from this drop.

## Slop signals
- [ ] README promises features that have no code — *not fired*: checked pages/canvas/agents/code-workspace features have real implementations
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — mostly inherited Zed (40 matches workspace-wide), only 1 in the Fanta crates
- [ ] Inconsistent architecture — *not fired*; modules follow one gpui/workspace pattern
- [x] No iteration history — single squashed commit in the download
- [ ] Copied upstream without attribution — *not fired*: GPL file + NOTICE.md + per-crate LICENSE-GPL duplicates + ARM Zed copyright header retained
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, thrifty licenses, collaborative agent design done right, honest alpha limitations; one slop signal (squashed history in the copy).

## Worth stealing
- **Design-as-source**: FNX text files (fanta.json/page.fnx/variables.json) so a design is Git-diffable — the .fig→editable-text path (`crates/fanta-fig-interop`, `crates/fanta-format`).
- **One design-op provider**: `crates/design_surface` consumed identically by built-in agent tools and a strictly-file-scoped live MCP socket (`fig_viewer/src/live_mcp.rs` — discovery file, atomic publish, pid check).
- **One-walk two-surface renderer**: `RasterRenderer::copy_rgba` (headless/tests) vs `render_to_canvas` (Metal present) behind the same `walk.rs` (`crates/fanta-render/src/raster`).
- **Doc-f64 / GPU-f32 precision split** documented in `ARCHITECTURE.md` §13 and executed in `transform.rs`.
- **Workspace item bootstrapping**: workspace_hooks observes worktrees and opens the canvas so folder opens, session restore and New Design all share one code path.

## Structural findings
1. **fig_viewer/src/view.rs is 23k LOC** — camera, tools, gestures, selection and 40+ actions in one entity; the fork concentrates its main complexity there. (alert)
2. **Zed fork inertia** — 920k LOC of intact upstream (editor 154k, project 97k, agent 90k) surrounds ~470k of Fanta work; fork maintenance cost is real. (info)
3. **120+-crate action surface** — fig_view alone declares 40+ gpui actions alongside panel-level action sets (design_panel, properties_panel); the command palette is the router. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| fig-viewer | [scan-fig-viewer.json](scan-fig-viewer.json) | [scan-fig-viewer.html](scan-fig-viewer.html) |
| fanta-render | [scan-fanta-render.json](scan-fanta-render.json) | [scan-fanta-render.html](scan-fanta-render.html) |
