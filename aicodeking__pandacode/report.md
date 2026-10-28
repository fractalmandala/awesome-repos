---
slug: aicodeking__pandacode
name: pandacode
github: https://github.com/aicodeking/pandacode
license: MIT (PandaCode attribution addition)
scanned: 2026-10-27
verdict: good
stack: [typescript, tauri, rust, sqlite, xterm, playwright]
---

# pandacode

> Tauri 2 agent workspace: plain TS UI, rust core, bots, VMs, GitHub view.

**Verdict: good** — ~865 `#[test]` fns (104 rust test files) + 75 TS test files, 40+ docs pages, deliberately framework-free webview UI; the surface is wide (bots, VMs, sandboxes, GitHub) but each is a contained rust module.

## What it is
PandaCode is a native desktop workspace (Tauri 2 + OS webview, Rust + SQLite, plain TypeScript — explicitly no React/bundled Chromium/Node server) that runs the coding CLIs you already have — Claude Code, Codex, Kimi Code, OpenCode, GLM Coding Plan, Xiaomi MiMo, plus Antigravity and Grok Build via ACP — side by side with projects, git, terminal and browser. Features: task board (Pending→Review), message queueing/steering, agent teams (architect + 3 workers), bots behind Telegram/Slack/Discord/WhatsApp/web, local VMs (Tart) and cloud sandboxes (Daytona/Cloudflare/AWS), simulators, approval-gated deploys, and a GitHub view with one-click review/fix/merge.

## Facts
| | |
|---|---|
| GitHub | https://github.com/aicodeking/pandacode (repository field) |
| License | MIT + attribution ("PandaCode License": public/commercial forks must state fork + link; LICENSE, package.json "SEE LICENSE IN LICENSE") |
| Stack | Tauri 2, Rust, SQLite, plain TS + CSS, xterm, marked/katex/dompurify, three.js (visuals), Playwright |
| Size | 125 TS files (22.2k) + 142 rust files (76.4k) + website/ |
| Activity | no git history in download; v0.3.5 |
| Tests | 75 TS test/spec files (node --test + playwright) + 104 rust files with ~865 `#[test]` fns; ci.yml + release.yml |
| Entry points | `src/main.ts` (webview), `src-tauri/src/lib.rs` (command registry) |

## Architecture
- **src/** — plain-TS UI modules per surface (agent-*, bots-*, architect, appearance, approval-bar, ...); `main.ts` (3,522 LOC) wires them to the bridge; `bridge.ts` is the invoke wrapper (documented in docs/ipc.md). [overview](scan-overview.html)
- **src-tauri/src** — rust core: providers/ per CLI incl. ACP, session/task controls, message_queue, turn_changes, context_recovery, conversation_fork/import, goal_state + slash_commands, vm/ + simulator/, cloud.rs sandboxes, github.rs/github_hub.rs/github_pull.rs, bots/, terminal.rs, usage_ledger + usage_keychain, secret_store.
- **website/** — docs site with its own playwright specs.
- **docs/** — 40+ topic pages (providers, task-board, architect-workers, bots, github, ipc...).

## Quality evidence
- Builds / type-checks: not run (tauri/cargo fetch needs network; `build: tsc --noEmit && vite build` verified present).
- Tests: ✅ 75 TS test/spec files + 104 rust files with ~865 `#[test]` fns; node --test + cargo test + playwright wired in package.json scripts.
- Structure: ⚠️ src/ is a flat module-per-surface layout with paired .ts/.css — disciplined but grows linearly; rust side is well-factored (one module per domain).
- Error handling: ✅ approval-gated deploys; secret_store + usage_keychain for credentials; approval-bar/modes on the UI side.
- Docs: ✅ README features each link a docs/*.md page; every README claim found a module (bots.ts ↔ bots docs, github.rs ↔ github docs, vm.rs ↔ VM docs).
- Dependencies: minimal TS runtime deps; three.js used for visuals; no telemetry claimed and none found.
- Maintenance: not checked (no git history); releases/ dir + set-version script + release workflow.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired*
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests both sides, disciplined module-per-domain rust core, honest custom license. 1 slop signal.

## Worth stealing
- **Framework-free webview UI** with per-surface .ts modules and a documented invoke surface (docs/ipc.md) — proof you don't need React for a Tauri workspace.
- **Bots-as-tasks**: five chat platforms adapt into ordinary agent tasks (`src-tauri/src/bots.rs`).
- **Goal state machines**: goal_state.rs + slash_commands.rs encode command workflows server-side.
- **Provider module pattern**: providers/ isolates each CLI incl. ACP bridging for Antigravity/Grok Build.
- **usage_ledger + usage_keychain**: usage accounting with credentials out of the DB.

## Structural findings
1. **src/main.ts is 3,522 LOC** of hand wiring — the flat UI module list is navigable but the entry file concentrates all bindings. (warn)
2. **rust core carries the weight** — 76k rust vs 22k TS; sessions/VMs/bots each isolated in their own module tree. (info)
3. **bots and VMs are ordinary tasks** — every integration funnels into the same task/composer/queue model. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
