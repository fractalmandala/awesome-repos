---
slug: julyx10__lap
name: lap
github: https://github.com/julyx10/lap
license: GPL-3.0-or-later
scanned: 2026-10-27
verdict: good
stack: [rust, tauri, vue, sqlite, c, cpp, objective-c]
---

# lap

> Tauri local-first photo manager: folder-first library, local AI search.

**Verdict: good** — 212 Tauri commands across 40 focused `t_*.rs` modules, 6 release pipelines (dmg/msi/deb/AppImage/homebrew/copr), 13 translated READMEs, and an honest privacy stance (local AI, no forced upload); the monolith is `t_sqlite.rs` at 10.9k.

## What it is
Lap is an open-source, local-first photo manager for macOS/Windows/Linux: browse family albums, find old photos fast, manage large libraries offline. Folder-first (works on your existing folders, no import lock-in), local AI (semantic search, similarity, smart tags, face features on-device), map and calendar views, dedup with review, montages, batch rename, backup. Vue 3 renderer over Tauri 2.

## Facts
| | |
|---|---|
| GitHub | https://github.com/julyx10/lap |
| License | GPL-3.0-or-later (LICENSE + src-tauri/Cargo.toml `license = "GPL-3.0-or-later"`) |
| Stack | Tauri 2, Rust, Vue 3, SQLite, C/C++/Obj-C shims (libraw, heif, jpeg, pasteboard), aptabase analytics |
| Size | 43 rust files (38.3k: t_sqlite 10.9k, t_utils 4.7k, t_cmds 3.8k) + 30 Vue/TS files (4.2k) |
| Activity | no git history in download |
| Tests | no dedicated test dir found in this drop; CI: pr-build, release (dmg/msi/deb/AppImage), release-homebrew, release-copr, release-windows, deploy-docs |
| Entry points | `src-vite/src/main.js` (webview), `src-tauri/src/main.rs` |

## Architecture
- **src-vite (Vue)** — views/ (library, map, calendar), components/ (40+: GridView, albums, dedup pane, dialogs, histogram, camera), three stores (config/library/ui). [overview](scan-overview.html)
- **src-tauri/src/t_*.rs** — one command domain per module: t_sqlite (library DB), t_cmds (general), t_image/jpeg/jxl/heif/raw/libraw (codecs via C/C++ shims), t_ai/t_ai_png/t_face/t_cluster/t_similar/t_lens (local AI), t_video/t_motion_photo/t_montage, t_storage/t_migration/t_batch_rename/t_dedup, t_protocol (thumbnail asset protocol), t_pasteboard/t_drag (per-OS), t_menu/t_config/t_startup.
- **i18n/** — 13 README translations; app locales in src-vite/src/locales.

## Quality evidence
- Builds / type-checks: not run (cargo/bun fetch not permitted; release workflows build all targets).
- Tests: ❌ no test directory in this drop (CI builds but does not run a test suite) — flagged.
- Structure: ✅ strict `t_` module convention (one domain per file); renderer small (4.2k) vs rust (38.3k).
- Error handling: not checked in detail.
- Docs: ✅ README claims (local-first, folder-first, local AI, no lock-in) map to modules; PRIVACY.md + THIRD_PARTY_NOTICES.txt present; docs/ site with deploy workflow.
- Dependencies: Tauri + image codec natives (libraw/heif/jxl shims vendored as source).
- Maintenance: not checked (no git history); trendshift badge + multi-platform releases suggest active use.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (THIRD_PARTY_NOTICES.txt)
- [ ] Doesn't build — *not fired* (not verified)
- [x] No automated tests — no test dir in the drop (manual/CI-build only)

## Verdict rubric
**good** — clean modular rust backend, real product polish (13 locales, 6 release channels), safety-conscious file handling; 2 slop signals (no tests seen, no history). Would be "ok" if the test absence reflects the repo rather than the drop.

## Worth stealing
- **t_ module convention**: one Tauri command domain per file keeps 212 commands navigable (`src-tauri/src`).
- **Custom thumbnail protocol**: t_protocol.rs serves image bytes efficiently to the webview.
- **Codec shims**: thin C/C++ files for libraw/heif/jpeg and per-OS pasteboards instead of heavy wrappers.
- **Folder-first storage**: index-on-top-of-folders rather than import-into-a-library (`t_storage.rs`).
- **13 README translations** for an indie app.

## Structural findings
1. **t_sqlite.rs is a 10.9k-LOC monolith** — all library DB access in one file; splitting per entity (photos/tags/faces/albums) is the next step. (warn)
2. **rust-side weight** — 38k rust vs 4k renderer; the UI is a thin client over well-partitioned commands. (info)
3. **no visible test suite** — CI builds and ships but doesn't run tests; local AI/dedup logic is the riskiest untested area. (warn)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
