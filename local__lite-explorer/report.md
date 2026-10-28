---
slug: local__lite-explorer
name: lite-explorer
github: unknown
license: MIT
scanned: 2026-10-27
verdict: good
stack: [typescript, svelte, sveltekit, tauri, rust, sqlite]
---

# lite-explorer

> Calm Tauri/SvelteKit file explorer: palette, transfer clipboard, S3/SMB.

**Verdict: good** — 526 rust `#[test]` fns + 72 TS test files, svelte-check + oxlint + clippy `-D warnings` scripts, rune-based state, and one founding feature done properly (persisted S3/SMB connections); no GitHub URL in the drop, so it's catalogued under a local slug.

## What it is
Lite Explorer is an experimental native-feeling desktop file browser (Tauri 2 + SvelteKit + TypeScript) with a macOS-inspired layout. Highlights: a ⌘⇧P command palette that finds commands, folder paths, quick-access places, favorites, locations and recent folders in one box; a transfer clipboard (⌘C/⌘X then ⌘V anywhere) with every long job (copies, moves, deletes, archives, uploads, downloads) tracked in a non-blocking Activity drawer filtered by All/Active/Done/Failed; and saved S3/SMB connections — the founding motivation was that Nicebucket didn't persist connections.

## Facts
| | |
|---|---|
| GitHub | unknown (README references nicebucket-org/nicebucket as prior art and spacedriveapp/spacedrive as inspiration — neither is this repo; no git remote) |
| License | MIT (LICENSE "© 2026 Litepod Studio"; desktop package.json + Cargo.toml `license = "MIT"`) |
| Stack | SvelteKit 2 + Svelte 5 (runes), TypeScript, Tauri 2 + Rust, Vite, oxlint, sqlite (app/db.rs) |
| Size | 768 TS/Svelte files (~50.1k LOC) + 121 rust files (41.6k): explorer 16.6k, preview 8.7k, network 5.9k, app 3.4k, remote 3.4k |
| Activity | no git history in download; v0.1.39, cliff.toml (git-cliff changelogs), changelog/ dir |
| Tests | 526 `#[test]` fns (rust) + 72 `*.test.ts` files; scripts: `test:ts` (bun test + vitest), `test:rust` (cargo test); CI: ci.yml, release.yml |
| Entry points | `apps/desktop/src/routes/+layout.svelte`, `apps/desktop/src-tauri/src/main.rs`, `apps/web` (Astro marketing site) |

## Architecture
- **apps/desktop/src/lib** (21 dirs) — file-pane controller, panes, tabs (rune store + tests), transfers + transfer-clipboard, remote locations (saved S3/SMB, network scan/status), quick-look, keyboard, virtual list, drag, selection, menu, settings. [overview](scan-overview.html)
- **apps/desktop/src-tauri/src** (8 modules, 41.6k) — explorer/ (entries, file_ops, archive, exif, favorites, watch, rename, transcode), preview/ (thumbnails), network/ (S3 buckets + SMB remote/transfer/write), app/ (window, menu, db, quicklook, swipe_nav, analytics), search, system, media. 154 `#[tauri::command]` fns.
- **apps/web** — small Astro site (index/privacy/terms).
- **State** — Svelte 5 runes in `.svelte.ts` files (command-palette, tabs, network-status, shortcuts-dialog); only devTools keeps a classic writable.

## Quality evidence
- Builds / type-checks: not run (bun/cargo fetch needs network; `check` script runs svelte-kit sync + svelte-check).
- Tests: ✅ 526 rust test fns + 72 TS test files; lint scripts enforce clippy `-D warnings`.
- Structure: ✅ clean module split on both sides; largest concern file counts are moderate (explorer 16.6k spread over ~20 files).
- Error handling: not checked in detail.
- Docs: ✅ README highlights map to code (palette ↔ command-palette.svelte.ts, transfer clipboard ↔ transfer-clipboard, drawer ↔ transfers, S3/SMB ↔ network/); ROADMAP.md, SECURITY.md, bilingual README.
- Dependencies: SvelteKit/Tauri mainstream; sqlite via rust.
- Maintenance: not checked (no git history); git-cliff + changelog dir imply release discipline.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`; no GitHub link found (local slug)
- [ ] Copied upstream without attribution — *not fired*
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real test mass on both sides, honest docs, one sharp founding feature; 1 slop signal.

## Worth stealing
- **Runes-everywhere state**: `.svelte.ts` rune stores per concern with colocated tests (`src/lib/state`, `src/lib/tabs`).
- **Transfer clipboard + Activity drawer**: non-blocking job queue with All/Active/Done/Failed filters — a good file-ops UX pattern.
- **Saved-connections persistence** for S3/SMB with network-status runes (`src/lib/remote`).
- **Rust module-per-domain** under src-tauri (154 commands over 8 dirs).
- **git-cliff changelog generation** from commits (cliff.toml + gen:changelog script).

## Structural findings
1. **lib/ is the whole app** (50k LOC over 21 dirs) — file-pane and remote are the two real subsystems; both carry their own tests. (info)
2. **explorer/ is the rust core** (16.6k) — file ops, watch, rename, transcode all server-side; the renderer stays thin. (info)
3. **Palette state is a single rune file** — command-palette.svelte.ts unifies commands/paths/favorites/recents; easy to extend, watch for kitchen-sink growth. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
