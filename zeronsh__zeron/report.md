---
slug: zeronsh__zeron
name: zeron
github: https://github.com/zeronsh/zeron
license: MIT
scanned: 2026-10-27
verdict: good
stack: [rust, gpui, loro, cloudflare-workers, workos, react, swift]
---

# zeron

> Rust gpui controller for coding agents: engine daemon, optional CRDT sync.

**Verdict: good** — 2,812 `#[test]` functions across 330 files, 12 CI workflows, an explicitly documented engine/UI/RPC split, and local-first sync designed as a frozen startup scope; the cost is a 242k-LOC ui crate.

## What it is
Zeron is a ground-up Rust rewrite of the original zeron Electron/Node controller for coding agents (Claude Code, Codex, ...). One protocol, three run modes: `zeron` (gpui app attaching to a detached `zeron-engine` daemon, embedding it as fallback), `zeron headless` (engine-only for VPS), and an optional synced mode where Loro CRDT docs mirror through Cloudflare Durable Objects (DeviceRoom/ChatRoom) with WorkOS auth, R2 attachments and APNs push — Postgres, the Hono server and WebRTC signaling were all deleted in the rewrite (ARCHITECTURE.md §1). 26 crates + the gpui app + an iOS app + a TS edge worker.

## Facts
| | |
|---|---|
| GitHub | https://github.com/zeronsh/zeron (README releases/sponsors links) |
| License | MIT (LICENSE "Copyright (c) 2026 Wing"; Cargo workspace `license = "MIT"`) |
| Stack | Rust 2021, gpui forked at zeronsh/zui rev 0dc69aa + gpui-component patch rev 6ffe5e5, Loro CRDT, tokio, Cloudflare Workers TS (edge/), Swift iOS app, Vite landing |
| Size | 655 rust files, 534k LOC total: ui 242k, engine 114k, harness 55k, client 17k, mobile 15k, sync 10.5k, proto 8.4k, text 8k, visual 6.2k, zatex 5.3k |
| Activity | no git history in download; version 0.2.119 |
| Tests | 2,812 `#[test]` fns in 330 files; 12 workflows (macos/windows/linux/release/testflight/ui-tests/voice-tests/preview-tests/cursor-compat/deploy...) |
| Entry points | `apps/zeron/src/main.rs` (headed), `crates/engine` (`zeron-engine` daemon + headless), `edge/src/index.ts` (Workers) |

## Architecture
- **apps/zeron** — headed app: attaches to the engine over localhost IPC (starting `zeron-engine` detached; reconnect backoff 1/2/5/15s; in-proc embed fallback). [overview](scan-overview.html)
- **crates/ui** (242k) — the gpui workspace: rail sidebar (attention-sorted sessions + spaces dropdown), device-local tabs (`ui-settings.json openTabs`), composer dock with dictation, files, comments, PR views, terminal, orb/voice stacks. [ui map](scan-ui.html)
- **crates/engine** (114k) — daemon backend: runner over the harness adapters, sessions/registry/journal, terminals, source control + change requests, workspace_host, doc_host (Loro), diff_sync, desktop notifications. [engine map](scan-engine.json)
- **crates/harness + client + proto/rpc** — agent adapter layer below the engine; typed RPC client the app uses everywhere.
- **crates/sync** — Loro mirror; sync is strictly opt-in and `WorkspaceScope` is frozen at engine start so a login can never silently swap the store (`auth.rs` vs `profile.rs`).
- **edge/** — Cloudflare Worker + DOs (DeviceRoom per device, ChatRoom per chat, R2 blobs, APNs, WorkOS JWKS).
- **crates/mobile + apps/ios** — Live Activities; voices crates (`voice*`) for dictation.

## Quality evidence
- Builds / type-checks: not run (cargo fetch not permitted; `rust-toolchain.toml` pinned).
- Tests: ✅ 2,812 inline `#[test]` across 330 files; CI matrix is unusually complete (macos, windows, linux-installer, release, testflight, ui/voice/preview tests, cursor compatibility).
- Structure: ⚠️ crisp crate seams (engine vs ui vs harness vs sync) but `crates/ui/src` is one flat 242k-LOC crate of 60+ sibling modules (pull_request_*, new_thread_background_*, composer_dock...).
- Error handling: ✅ 0 TODO/FIXME matches across all crates; anyhow/gpui Result style throughout (no unwrap census run).
- Docs: ✅ ARCHITECTURE.md is a real design doc (topology, headed/headless, auth-vs-scope table, cloud checkout design, research notes in docs/research/); README claims (multi-device, headless, local default) all map to code.
- Dependencies: gpui pinned to an in-house fork with an explanatory comment; edge is package-lock'd TS with vitest configs for workerd.
- Maintenance: not checked (no git history in download); CHANGELOG absent, version 0.2.119 suggests fast iteration.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (THIRD_PARTY_NOTICES.md + NOTICE for forks)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, deliberate architecture, honest docs, 1 slop signal.

## Worth stealing
- **Engine/viewport split with one RPC**: same protocol in-proc, daemon, and remote (managed.rs reconnect ladder + embed fallback is documented decision-making).
- **Frozen WorkspaceScope**: auth state changes never re-resolve the open store — a local-first guarantee encoded in types (`profile.rs`).
- **CRDT-first sync**: Loro docs hosted by the engine and mirrored by a thin DO relay; Postgres and signaling deleted, not wrapped.
- **Notifications owned by the engine**: banners keep firing with the app closed; focus is reported by the UI (`SetViewportFocus`).
- **Reviewable agent changes**: change_request_store + review threads in the engine, diffs surfaced in the UI.

## Structural findings
1. **crates/ui is a 242k-LOC flat crate** — 60+ sibling modules; sub-crate grouping (composer, pr, files, voice) is the next structural lever. (warn)
2. **engine owns everything long-lived** — sessions, terminals, docs, notifications all live in the daemon; the UI restarts freely without losing state (run_journal replays). (info)
3. **auth/scope separation** — login/logout can never re-resolve storage; the synced profile requires a restart-borne scope. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| engine | [scan-engine.json](scan-engine.json) | [scan-engine.html](scan-engine.html) |
| ui | [scan-ui.json](scan-ui.json) | [scan-ui.html](scan-ui.html) |
