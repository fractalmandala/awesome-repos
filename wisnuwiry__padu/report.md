---
slug: wisnuwiry__padu
name: padu
github: https://github.com/wisnuwiry/padu
license: GPL-3.0-only
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, drizzle, sqlite, typescript, react, vite]
---

# padu

> Native Rust/GPUI desktop for local coding agents with a driver core.

**Verdict: good** — 1,176 `#[test]` fns in the Rust workspace, an I/O-free versioned protocol crate shared by Rust and TS clients, per-provider drivers isolated in one dir; young project (alpha) with the weight concentrated in apps/desktop (104k).

## What it is
Padu is a native desktop app (Rust + zed's GPUI; macOS/Windows/Linux, signed installers) for working with local coding agents — Amp, Claude Code, Codex CLI, Cursor CLI, DeepSeek and ACP agents — keeping projects, sessions and transcripts entirely on your machine. A workspace: apps/desktop (GPUI app), crates/padu-{core,protocol,client,mermaid,daemon}, TS web/landing/mobile sharing a TS protocol client, and a drizzle/SQLite db for the web side. Optional `padu-daemon` keeps sessions alive after the app quits.

## Facts
| | |
|---|---|
| GitHub | https://github.com/wisnuwiry/padu |
| License | GPL-3.0-only (LICENSE GPL text; apps/desktop/Cargo.toml `license = "GPL-3.0-only"`) |
| Stack | Rust 1.83+, GPUI, rust-i18n (en/zh-CN/ja/id), drizzle + SQLite (web), TS/React/Vite clients |
| Size | 234 rs files, 182k LOC (desktop 104k, core 57k, protocol 10.4k, client 6.2k, mermaid 2.9k, daemon 1.3k) + 23k TS |
| Activity | no git history in download; v0.x alpha, CHANGELOG present |
| Tests | 1,176 `#[test]` fns (136 files); 47 TS test files; CI: test.yml, release.yml, sync-release.yml, deploy-*.yml |
| Entry points | `apps/desktop/src/main.rs` → `padu::run()`; `crates/padu-daemon/src/main.rs`; web apps |

## Architecture
- **apps/desktop** — GPUI app; app.rs (4,010 LOC) entity + 33 app/ modules (composer, board, branches, command_palette, right_panel, dialogs, drafts, onboarding). [overview](scan-overview.html)
- **crates/padu-core** — domain: driver/ (claude, codex, amp, acp, cursor, deepseek, computer_use) + one session file per provider, checkpoints, git_branch, audit, blob_store, frontmatter. [core is folded into the overview map]
- **crates/padu-protocol** — serializable wire types only, no I/O; carries the i18n registry (en/zh-CN/ja/id).
- **crates/padu-client + packages/padu-client** — Rust and TS clients over the same protocol; web/mobile share the TS one.
- **crates/padu-daemon** — background daemon the app hands live sessions to on quit.
- **crates/padu-mermaid** — mermaid rendering for transcripts.
- **db/ (drizzle)** — schema + migrations for the web/dashboard store.

## Quality evidence
- Builds / type-checks: not run (cargo/npm fetch not permitted; `RELEASING.md` + CI present).
- Tests: ✅ 1,176 rust test fns; 47 TS tests; CI test.yml.
- Structure: ✅ clean crate boundaries; protocol I/O-free; desktop is the heavy one (104k, biggest file app.rs 4,010).
- Error handling: not checked in detail (no unwrap census run); anyhow-style results in core.
- Docs: ✅ README (architecture section, install per platform, agent list) matches code; AGENTS.md, NOTICE.md, SECURITY.md, RELEASING.md present.
- Dependencies: gpui from zed; drizzle on the TS side; small dep surface.
- Maintenance: active development stated in README (alpha); no git history in this drop.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (NOTICE.md, THIRD-PARTY refs)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, disciplined protocol layer, per-provider isolation; alpha status honestly declared. 1 slop signal.

## Worth stealing
- **I/O-free protocol crate** shared by Rust and TS clients (`crates/padu-protocol/src/lib.rs`).
- **Driver/session duality**: one driver module + one session file per provider; adding provider #8 is mechanical.
- **Optional daemon**: `padu-daemon` for sessions-outliving-the-app, driven by an explicit hand-off.
- **i18n in the protocol**: locale registry shipped with wire types so translations rebuild with the shared crate.
- **Checkpoint module** for agent file-edit rollback (`crates/padu-core/src/checkpoint.rs`).

## Structural findings
1. **apps/desktop is 104k LOC** — app.rs 4,010 + 33 modules; extracting board/panels into sub-crates would keep it navigable. (warn)
2. **protocol-first design** — wire types have zero I/O, letting TS and Rust clients consume one contract. (info)
3. **session-per-provider files** — easy to audit provider-specific behavior; audit.rs watches them all. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
