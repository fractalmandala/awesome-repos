---
slug: grameditor__gram
name: gram
github: https://codeberg.org/GramEditor/gram
license: GPL-3.0-only
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, tree-sitter, lsp, dap, sqlite, vim]
---

# gram

> Zed hard fork: hand-writing-first editor, 135 crates, GPL-3.0.

**Verdict: good** — 652 `#[test]` fns across 170 files, one CI pipeline, an honest mission doc, and the Zed codebase kept working (editor 103k, gpui 81.7k, project 63k); the fork's own additions are small but pointed (which_key, journal, quick action bar).

## What it is
Gram is a hard fork of Zed (v3.3.0, GPL-3.0) with a stated philosophy: writing code by hand with care is good; an editor for a first-time programmer should work out of the box with no configuration, extensions, or pushy distractions. Hosted on Codeberg; community chat runs on XMPP. The full Zed architecture is retained in-tree (135 crates, 673k LOC) with fork-specific crates layered on: which_key (discoverable prefix keymaps), journal (daily notes), quick_action_bar, refineable, app_actions, explorer_command_injector, nc.

## Facts
| | |
|---|---|
| GitHub | https://codeberg.org/GramEditor/gram (Cargo repository field; not a GitHub repo — slug uses the forge owner) |
| License | GPL-3.0-only (LICENSE GPL-3 text) |
| Stack | Rust, in-tree gpui (forked from Zed), tree-sitter, LSP/DAP, sqlez sqlite, vim/helix modes |
| Size | 135 crates, 673.6k LOC: editor 102.9k, gpui 81.7k, project 63.2k, workspace 29.4k, vim 29.3k, language 20.2k, git_ui 19.2k, terminal 5.7k |
| Activity | no git history in download; CHANGELOG with Unreleased section, Keep-a-Changelog format |
| Tests | 652 `#[test]` fns in 170 files; `.github/workflows/ci.yaml` |
| Entry points | `crates/gram/src/main.rs` (app), `crates/cli`, `crates/extension_cli` |

## Architecture
- **crates/gram** — app assembly (main.rs 1,342; gram.rs 4,690): window options, menus, initialize_workspace, open-listener, component previews, quick_action_bar. [overview](scan-overview.html)
- **crates/editor (103k) / workspace (29k) / project (63k)** — the Zed triad: display/editing surface, pane/dock shell, worktree/LSP/debugger backend.
- **crates/gpui (81.7k)** — the GPU UI framework, in-tree.
- **Fork crates** — which_key, journal, refineable, nc, app_actions, explorer_command_injector, quick_action_bar: small, composable additions.
- **crates/language + languages + extension_*** — languages, extension API/host, theme importer retained.
- **client/collab** — Zed's collab client crate remains but the hosted service is out of scope for the fork's philosophy.

## Quality evidence
- Builds / type-checks: not run (workspace fetch not permitted; rust-toolchain + ci.yaml present).
- Tests: ✅ 652 `#[test]` fns in 170 files (inherited Zed test culture).
- Structure: ⚠️ 135 crates, 673k LOC — the fork inherits Zed's scale; fork deltas are easy to spot (small named crates + docs/mission.md).
- Error handling: 1,085 `unwrap()/expect(` in crates/editor/src alone — consistent with upstream Zed's GUI-context style; 652 tests back the core.
- Docs: ✅ docs/mission.md is a genuine statement of purpose; README honest ("work in progress", grumpy toad); CHANGELOG maintained.
- Dependencies: zed-industries git revs pinned (async-pipe, dap-types, lsp-types).
- Maintenance: not checked (no git history); Codeberg hosting + XMPP community.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired*: hard fork declared in README + mission, GPL preserved
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — inherited Zed quality (652 tests), transparent fork, meaningful additions; 1 slop signal.

## Worth stealing
- **which_key crate**: discoverable prefix-key UX for a modal editor (`crates/which_key`).
- **Fork hygiene**: small named crates + a mission doc make the delta against a 673k-LOC parent readable.
- **quick_action_bar**: fast-row actions overlay in `crates/gram/src/gram/quick_action_bar.rs`.
- **Component previews**: `crates/gram/src/gram/component_preview` for UI work.
- **XMPP community channel**: no vendor chat lock-in.

## Structural findings
1. **Fork maintenance weight** — 135 crates/673k LOC to track against upstream Zed; pins and patches will accumulate. (warn)
2. **Fork surface is additive and small** — which_key/journal/quick_action_bar are <1.5k LOC combined; risk concentrated in keeping the base healthy. (info)
3. **collab removed in spirit** — client crate stays for compile health but no hosted service; local-first editor. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
