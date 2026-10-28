---
slug: anzal1__panoply
name: panoply
github: https://github.com/anzal1/panoply
license: MIT OR Apache-2.0
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, wasm, mcp, ffmpeg]
---

# panoply

> Rust/GPUI file-utility suite: command bar intent, 23 tool crates, MCP.

**Verdict: good** — 2,063 `#[test]` fns, 8 CI workflows incl. desktop journeys/smoke, a safety-first design (results are new files, Undo→Trash), and tool crates that are genuinely separable; the vendored gpui-base patch is the only wart.

## What it is
Panoply is "every file utility in one open-source app": drop a file on a command bar, type "make this under 50 kb", press Return — the best local tool runs and the result lands as a new file next to the original (Undo moves it to Trash; nothing is uploaded). Cross-platform Rust/GPUI desktop (macOS/Windows/Linux installers) over 23 crates: image (24.8k), media (23.6k), pdf (11.9k), ocr (9.7k), vision, find/disk/id, share, tts, archive, convert, plus `panoply mcp` exposing every tool to Claude/Cursor/VS Code agents, a headless CLI, and sandboxed wasip2 tool modules.

## Facts
| | |
|---|---|
| GitHub | https://github.com/anzal1/panoply (Cargo repository field) |
| License | MIT OR Apache-2.0 (LICENSE-MIT + LICENSE-APACHE; workspace `license = "MIT OR Apache-2.0"`) |
| Stack | Rust 2024 (1.88), GPUI (with in-repo patched gpui-base), wasm32-wasip2 tool modules, MCP server, ffmpeg-backed media |
| Size | 691 rs files, 293.8k LOC — desktop 27.1k app + ~30k vendored patch; image 24.8k, media 23.6k, pdf 11.9k, ocr 9.7k, share 9.5k, id 8k, vision 7.8k, host 6.8k |
| Activity | no git history in download; v0.3.0-beta |
| Tests | 2,063 `#[test]` fns; CI: ci.yml, desktop-journeys.yml, desktop-smoke.yml, installers, release, site, update-manifests |
| Entry points | `apps/desktop/src/main.rs`, `crates/panoply-cli` (`panoply` binary incl. `panoply mcp`) |

## Architecture
- **apps/desktop** — GPUI app: ui/ (app, bar, panel, disk, chrome, live actions), intent.rs (words→size/convert/search intent), jobs.rs (tool job queue), catalog.rs, history.rs, selfcheck.rs, disk_review, share. [overview](scan-overview.html)
- **crates/panoply-core** — shared tool traits/registry every tool crate plugs into; jobs and MCP both call through it.
- **Tool crates** — image/media/pdf/ocr/vision/find/disk/id/share/tts/archive/convert; platform-macos/-windows + host + ffi for native codecs.
- **crates/panoply-wasm + modules/** — sandboxed wasip2 tool modules (template-rust, examples, killtest) built as standalone cargo projects.
- **crates/panoply-mcp + panoply-cli** — `panoply mcp` serves the catalog to agents; the CLI does headless runs.

## Quality evidence
- Builds / type-checks: not run (workspace fetch not permitted; `cargo check` scripts in CI).
- Tests: ✅ 2,063 test fns; desktop-journeys + desktop-smoke CI exercise the real app.
- Structure: ✅ one-crate-per-tool-domain; desktop app files mostly 500–2.5k LOC (largest ui/panel.rs 2,578).
- Error handling: ✅ results-as-new-files + Undo-to-Trash policy; selfcheck.rs verifies tool availability at runtime.
- Docs: ✅ README claims (command bar, no overwrite, MCP for agents) all map to code; CHANGELOG, THIRD-PARTY-NOTICES, SECURITY, SUPPORT present.
- Dependencies: mostly rust-ecosystem; gpui-base patch vendored in-repo with fixes (input/text/dock).
- Maintenance: not checked (no git history); site-on-release automation implies active shipping.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [x] Large vendored patch — `apps/desktop/patches/gpui-base` (~30k LOC) carried in-repo (upstream fixes awaiting release)
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (patch dir is declared, THIRD-PARTY-NOTICES present)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — safety-first product design, real test mass and CI journeys, clean crate seams; 2 slop signals, both benign (vendored patch + missing history).

## Worth stealing
- **Results-as-new-files + Undo→Trash** as the default mutation policy for any file tool (`apps/desktop/src/jobs.rs`).
- **Small intent parser over fuzzy search**: three understood intents, everything else fuzzy tool search (`apps/desktop/src/intent.rs`).
- **One catalog, three consumers**: GPUI bar, headless CLI and MCP server all drive the same panoply-core registry.
- **wasip2 tool modules**: sandboxed, standalone-cargo tool modules with a template and a killtest harness.
- **Desktop journeys CI**: real-app journey tests, not just unit tests.

## Structural findings
1. **Desktop app is lean, crates are heavy** — the 27k app orchestrates ~250k of tool crates; each tool domain is independently testable. (info)
2. **Vendored gpui-base patch (~30k)** — input/text/dock fixes carried in-tree until upstream merges; upgrade friction lives here. (warn)
3. **MCP/CLI share the desktop's core** — agents and scripts get identical tool semantics to the GUI. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
