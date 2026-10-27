---
slug: hardbeat920__monocode
name: monocode
github: https://github.com/hardbeat920/monocode
license: MIT
scanned: 2026-10-27
verdict: good
stack: [typescript, react, tauri, rust, node, vitest, playwright, vite]
---

# monocode

> Tauri GUI for local coding agents: sessions, monos, host service.

**Verdict: good** — 869+ test files (~4,841 TS test cases), zero @ts-ignore, and a single provider-agnostic harness protocol shared by app, host and mobile; the price is two monoliths (src/app/App.tsx 15k, src-tauri/src/fs.rs 9.7k).

## What it is
MonoCode is a cross-platform (macOS/Linux/Windows) desktop GUI, built with Tauri, that runs your already-installed coding agents — Claude Code, Codex, Cursor, Grok Build, OpenCode, Antigravity, Pi, omp, fx, Hermes, Devin — as tabs over your existing subscriptions. Each tab is a session; split panes run sessions side by side; the composer is the unified input; `/operator` hands a mono agent the ability to open sessions and manage worktrees and notes. A Node host service (`host/`) runs headless so a phone (`mobile/` + `crates/mobile-link`) can drive the same sessions remotely over `/rpc`; the Rust core (`src-tauri/`, 57.7k LOC, 308 Tauri commands) owns fs, agent spawning, PTY terminals, git worktrees and checkpoints.

## Facts
| | |
|---|---|
| GitHub | https://github.com/hardbeat920/monocode (README release links) |
| License | MIT (LICENSE "Copyright (c) 2026 Nick"; no package.json license field — package is `"private": true`) |
| Stack | TypeScript 5 + React, Tauri 2 + Rust 2021 core, Node host (ESM only stdlib deps), Vite, vitest + Playwright, 11 provider adapter TS modules under `src/integrations/harness/providers/` |
| Size | src/ 1,181 TS files (~331k LOC), host/ 18 source files (~6k), src-tauri 59 rust files (57.7k), crates/mobile-link Swift+Rust pairing, mobile/ companion app |
| Activity | no git history in download; CHANGELOG is 174k of release entries (v0.16.0 current) |
| Tests | 869 `*.test.ts(x)` files in src (~4,841 vitest cases counted via `test(`+`it(`), 24 host test files with its own vitest config; `npm run check` = vitest + tsc + clippy `-D warnings` + cargo test |
| Entry points | `src/main.tsx` (Tauri webview), `host/cli.ts` (headless host), `src-tauri/src/lib.rs` (Rust plugin registration) |

## Architecture
- **src/main.tsx → src/app/App.tsx** — Tauri webview entry; App.tsx (15,062 LOC) is the workspace shell: workspaces, tabs, split panes, keybindings, window events.
- **src/features/<name>/** — 21 feature packages; the giants are sessions 68k LOC, inbox 20k, monos 17k, files 17k, settings 15k, source-control 15k. Each has model/ui/data with collocated tests. [global map](scan-overview.html) [sessions](scan-sessions.html)
- **src/integrations/harness/** — one provider-agnostic event protocol (`core/apply`) + 11 provider adapters (claude, codex, cursor, devin, fx, grok, hermes, omp, opencode, pi, antigravity); both the Tauri process and the Node host decorate that same protocol.
- **host/** — headless Node service: server.ts `/rpc` + `/lifecycle` (no WS, chunked streams by batches of 120ms), engine.ts applies harness events to the store, workspace-commands.ts surfaces file/diff/git verbs, service.ts installs a launchd/Windows wrapper. [host map](scan-host.html)
- **src-tauri/** — rust core; `fs.rs` (9,721 LOC) is the file boundary, `harness.rs` (4,319) spawns/orchestrates agents, `mcp.rs` serves a local MCP, `checkpoint.rs` snapshots file edits for rollback, `session_store.rs` persists sessions. [src-tauri map](scan-src-tauri.html)
- **mobile/ + crates/mobile-link** — pairing surface; the phone leans on `host` for streams and upload/download of attachments.

## Quality evidence
- Builds / type-checks: not run (`npm ci`/cargo fetch would need network; scripts confirmed: `build: tsc && vite build`, `check:web` includes `tsc --noEmit`, `check:rust` includes clippy `-D warnings`).
- Tests: ✅ 869 src test files (~4.8k cases) + 24 host tests + Playwright browser suite in `playwright.config.ts`, enforced in `npm run check`.
- Structure: ⚠️ two monoliths: src/app/App.tsx 15,062 LOC and src-tauri/src/fs.rs 9,721 LOC; the rest stays modular (features 300–2k LOC per file, small file count).
- Error handling: ✅ 0 `@ts-ignore|@ts-expect-error`; 6 TODO/FIXME/HACK mentions across 331k LOC; 25 `expect(` occurrences in the Rust core (152 test files ship their own assertions).
- Docs: ✅ README claims map to code (agents list ↔ `providers/` 11 adapters, `/operator` ↔ monos, worktrees ↔ `git-worktrees.ts`, docs/providers.md exists); NOTICE, SECURITY.md, CONTRIBUTING present.
- Dependencies: ✅ zero runtime dependencies in the web package (private:true, dev tooling only); host ships a vendored stdlib-only service; Rust core uses tokio+tauri.
- Maintenance: not checked (no git history in download); CHANGELOG shows 174k of release notes sponsored at dl.usemono.dev + cc release configs per platform.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*; every feature keeps model/ui/data layering
- [x] No history in download — single zip, single squash
- [ ] Copied upstream without attribution — *not fired*
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, layered features, protocol reuse across app/host/mobile; 1 slop signal (no history in download).

## Worth stealing
- **harness/core as a boundary**: provider-agnostic event apply (`applyHarnessEvent`) that all three runtimes import — no CLI-specific logic outside `src/integrations/harness/providers/`.
- **rpc-only host**: `/rpc` + chunked streams in `host/server.ts` proved a lean fit for mobile; no websocket stack, catalog invalidation by binary mtime.
- **Bounded transcript rendering**: `AgentTranscript.tsx` focused on pacing/folding with 13 micro test files (bubbleWidth, firstPaint, promptRise...) — a render-focused test culture.
- **FS surface parented in one file**: `src-tauri/src/fs.rs` at 9.7k acts as the single-file seam where path safety is tested and split can start.
- **Checkpoints**: `checkpoint.rs` snapshot ID tracked with each turn; rust owns writes while TS only maps semantics.

## Structural findings
1. **App.tsx is a 15k-LOC shell** — src/app/App.tsx holds workspaces, tab/panes, keybindings and window events in one entity while `src/app/shell/` files stay tiny; extraction into features is the top lever. (alert)
2. **fs.rs is the 9.7k-LOC rust boundary** — 300 of the 308 Tauri commands route file watcher/diff/snapshot jobs through it before any real splitting of other crates; sessions UI calls it from many features. (warn)
3. **test culture carries the repo** — 869 test files (mostly 1:1 next to source) plus 24 host tests under a distinct vitest config make semantics self-checking; `npm run check` is a one-command gate. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| host | [scan-host.json](scan-host.json) | [scan-host.html](scan-host.html) |
| sessions | [scan-sessions.json](scan-sessions.json) | [scan-sessions.html](scan-sessions.html) |
| src-tauri | [scan-src-tauri.json](scan-src-tauri.json) | [scan-src-tauri.html](scan-src-tauri.html) |
