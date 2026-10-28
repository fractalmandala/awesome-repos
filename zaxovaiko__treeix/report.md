---
slug: zaxovaiko__treeix
name: treeix
github: https://github.com/zaxovaiko/treeix
license: Apache-2.0
scanned: 2026-10-27
verdict: good
stack: [rust, typescript, react, tauri, pty, mcp, git]
---

# treeix

> Tauri review workspace: worktree diffs, PRs, tickets, durable agent PTYs.

**Verdict: good** — 228 `#[test]` fns, CI + release workflows, a mockable typed IPC layer, and one clear product loop (diff → note → agent's next prompt); small (52k LOC) and cohesive.

## What it is
Treeix is a Mac/Linux Tauri app that puts a worktree, its diff, review comments, its pull request and the agent session that produced it on one screen — review notes you write on selected diff lines are queued back to the Claude Code / Codex session in that worktree as its next prompt. A background daemon (`crates/ptyd`) owns every shell and agent session so they survive app quits with scrollback; a browser pane on the system webview (with console/network) is drivable by agents through the app's MCP server; Packages view shows a module graph with unresolved imports; tickets and routines round it out.

## Facts
| | |
|---|---|
| GitHub | https://github.com/zaxovaiko/treeix |
| License | Apache-2.0 (LICENSE + workspace `license = "Apache-2.0"`) |
| Stack | Rust 2024 (1.90), Tauri 2, React/TS renderer, custom ptyd daemon, MCP server, git |
| Size | 88 rs files (31.3k: ptyd 10.7k, core 10.4k, src-tauri 9.1k, proto 1.2k) + 142 TS files (21.1k) |
| Activity | no git history in download |
| Tests | 32 test/spec files incl. ipc mock transports; ci.yml + release.yml |
| Entry points | `apps/desktop/src/main.tsx`, `apps/desktop/src-tauri/src/main.rs`, `crates/ptyd` |

## Architecture
- **apps/desktop** — Tauri app: renderer features (agents, browser, graph, mcp, plans, routines, tickets, voice), panels (Changes/Explorer/History/Prs/Tickets), panes, shell. [overview](scan-overview.html)
- **apps/desktop/src/ipc** — typed bindings with mock transports (`mockAgents.ts`, `mockBrowser.ts`, `mockChat.ts`...) so the whole UI tests without processes; `tauri.ts` is the real one.
- **apps/desktop/src-tauri** — Tauri commands: browser (cookies/inject.js), daemon, commands, menu, tray, quick, voice, updater, service.
- **crates/core** — domain: agents, git(+graph), plans, prs, routines, tickets, mcp.rs server, secrets, settings, search, tailscale.
- **crates/ptyd** — the durability daemon: owns shells and agent sessions (chat/, hooks.rs, clients) independent of the UI.

## Quality evidence
- Builds / type-checks: not run (cargo/bun fetch needs network; `rust-toolchain.toml` pinned at 1.90).
- Tests: ✅ 32 test/spec files; mock IPC makes UI logic testable in isolation; CI + release workflows present.
- Structure: ✅ small crate set with clear ownership (ptyd owns processes, core owns domain, tauri owns OS); renderer organized features/panels/panes/shell.
- Error handling: not exhaustively checked; rust core uses typed errors per module (secrets.rs, settings.rs).
- Docs: ✅ README claims map 1:1 to code (line-note queue, durable sessions via ptyd, browser pane via system webview + MCP, packages graph); docs/architecture.md exists.
- Dependencies: lean; Tauri + git; no heavyweight UI kit.
- Maintenance: not checked (no git history); release + CI workflows present.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (NOTICE present)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — cohesive product loop, real tests with mockable IPC, clean daemon ownership. 1 slop signal.

## Worth stealing
- **Mockable IPC seam**: `apps/desktop/src/ipc` ships typed bindings plus mock transports so every feature tests headlessly.
- **Process ownership by a daemon**: ptyd owns PTYs/agent sessions; the UI is disposable (same idea as zeron's engine, smaller).
- **Review notes as prompts**: diff-line selection → queued prompt is a tight human-in-the-loop design.
- **Browser pane driven by MCP**: system webview + mcp.rs lets agents click/type with console/network visible.
- **Module graph view** over unresolved imports (crates/core/src/graph).

## Structural findings
1. **ptyd is the spine** — sessions, hooks and clients live there; a ptyd crash is the only way to lose running work. (warn)
2. **Review loop is the product** — Changes panel + note queue is the main flow; everything else (tickets, routines, PRs) hangs off it. (info)
3. **Mock IPC doubles as a contract** — renderer can't drift from the rust side without breaking the mock/type pair. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
