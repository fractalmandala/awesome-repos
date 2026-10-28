---
slug: befeast__okilum
name: okilum
github: https://github.com/BeFeast/okilum
license: MIT
scanned: 2026-10-27
verdict: good
stack: [rust, gpui-kit, sqlite, mcp, forgejo]
---

# okilum

> gpui-kit Markdown vault app: reader shell, cored sidecar, AI Brain.

**Verdict: good** — 2,480 `#[test]` fns across 401 files, a frozen JSON-lines sidecar protocol (`okilum-cored`, docs/PROTOCOL.md) that decouples any second client, and an AI Brain designed so "markdown stays the durable product data"; 15 Forgejo CI workflows.

## What it is
Okilum is a fast native desktop app for folders of Markdown notes (no account/server/import; caches are rebuildable). Reads the whole vault: wiki-links + backlinks with context, frontmatter properties, callouts, wide tables, code labels, images/SVG, native `.excalidraw.md` drawings; edits with atomic saves, note history, and rename/move that rewrites every link. A 10-crate Rust workspace on a vendored gpui-kit: okilum-shell (170k, the reader/editor UI), okilum-core (66.8k vault engine), okilum-brain (55.5k AI Brain: chat, proposals, retrieval, t3, Todoist, Maestro, inbox), okilum-cored (JSON-lines sidecar + MCP), sync trio, paste, http, plus a JS web inbox and Forgejo-based CI (GitHub mirror workflows included).

## Facts
| | |
|---|---|
| GitHub | https://github.com/BeFeast/okilum (CONTRIBUTING bug-report link) |
| License | MIT (LICENSE "Copyright (c) 2026 Oleg Kossoy"; about.toml + licenses/ dir) |
| Stack | Rust (vendored gpui-kit clone as its own workspace, path-depended), SQLite, JSON-lines protocol, MCP, Forgejo CI |
| Size | 650 rust files, 345.4k LOC: shell 170.6k, core 66.8k, brain 55.5k, sync-controller 18.9k, t3 10k, paste 5.5k |
| Activity | no git history in download (zip); extracted from okilum-main.zip |
| Tests | 2,480 `#[test]` fns in 401 files (colocated `*_tests.rs` in brain/core/shell); web/inbox JS tests; 15 CI workflows (ci, brain-ci, commit-ci, excalidraw-compat, linux-release, github-mirror...) |
| Entry points | `crates/okilum-shell/src/main.rs` (desktop), `crates/okilum-cored/src/main.rs` (sidecar `index|mcp`), `web/inbox` |

## Architecture
- **okilum-shell (170k)** — the gpui-kit app: ~40 `reader_*` modules (editor, embeds, drawings, outline, links, history, occlusion, incremental loading, feedback, diagnostics), projects (picker, avatar, icons), quick_open, pdf_engine, quicklook, platform, app menus. [overview](scan-overview.html)
- **okilum-core (66.8k)** — vault engine: document_links, link_candidates/registration/rewrite, note_move, excalidraw, analyzer, IR, facets, callouts, delimited, export, archive, file_editor, deep_link, json_view, maestro_observation.
- **okilum-cored** — frozen JSON-lines sidecar (`index|mcp --vault PATH`); "any second client reaches the vault through this frozen protocol rather than a private API".
- **okilum-brain (55.5k)** — AI backend: chat, discussion_send, proposals, retrieval (+readiness tests), t3 (+recovery/routes/compat), todoist picker, maestro control, inbox_plan, runtime Adapter for provider transports.
- **sync trio + web/inbox** — sync-controller/supervisor/sync with a JS pairing UI (forgejo/publication forms, service worker).
- **vendor/** — gpui-kit clone fetched by scripts/vendor-setup.sh (not present in the zip; excluded from the workspace).

## Quality evidence
- Builds / type-checks: not run (vendored gpui-kit must be fetched via scripts/vendor-setup.sh; network needed).
- Tests: ✅ 2,480 `#[test]` fns in 401 files — the largest test mass of anything mapped so far relative to size.
- Structure: ✅ strong crate separation; shell uses strict `reader_*`/`*_tests.rs` conventions; brain separates runtime/Adapter from product data.
- Error handling: not checked in detail; t3_recovery + conflict-style modules suggest durability focus.
- Docs: ✅ docs/ has PRD.md, PROTOCOL.md, RENDER/SEARCH-ACCEPTANCE.md, ai-brain-alpha.md — spec-driven development artifacts; README claims (whole-vault reads, undoable edits, relink on rename) map to core modules.
- Dependencies: vendored gpui-kit; everything else Rust-ecosystem.
- Maintenance: not checked (no git history); 15 CI workflows incl. excalidraw compatibility and inbox CI.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git` (and vendor/ is a setup script fetch, not in the zip)
- [ ] Copied upstream without attribution — *not fired* (THIRD_PARTY_NOTICES + licenses/)
- [ ] Doesn't build — *not fired* (not verified; vendor fetch required)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — exceptional test mass, protocol-driven second-client design, spec docs; 1 slop signal (no history; vendor fetch caveat).

## Worth stealing
- **Frozen sidecar protocol**: any second client (webview, MCP adapter, scripts) talks JSON-lines to okilum-cored instead of private APIs — with a written PROTOCOL.md and acceptance docs.
- **"Markdown stays durable"**: Brain runtime plugs provider transports behind an Adapter; execution DBs never become product data (`crates/okilum-brain/src/lib.rs`).
- **Rename-with-relink**: note_move + link_rewrite keep the link graph honest across moves.
- **Colocated `*_tests.rs`** in every brain/core/shell service.
- **Spec-driven docs**: PRD + protocol + render/search acceptance criteria as first-class repo files.

## Structural findings
1. **okilum-shell is 170k LOC** — even with strict reader_* conventions, one crate holds the entire UI; feature sub-crates are the next cut. (warn)
2. **cored is the integration spine** — shell and every external client depend on the same frozen protocol; changes are deliberately versioned. (info)
3. **Brain ≠ database** — provider state isolated behind runtime Adapter + Snapshot; vault files remain the source of truth. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
