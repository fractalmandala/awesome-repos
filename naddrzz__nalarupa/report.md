---
slug: naddrzz__nalarupa
name: nalarupa
github: https://github.com/naddrzz/nalarupa
license: MIT
scanned: 2026-10-27
verdict: good
stack: [rust, react, gpuix, mcp, claude]
---

# nalarupa

> Infinite-canvas vector editor with AI agents: GPUIX React over rust core.

**Verdict: good** — ~190 rust `#[test]` fns, a documented MCP surface shared by built-in and external agents, and a careful agent-undo model (streams commit as one undo step); young project with thin app-side tests and 7 always-on CI workflows including Claude-driven ones.

## What it is
Nalarupa ("reason given form") is a native infinite-canvas design editor with AI agents: a Rust core (document, engine, GPU render pipeline, .pen documents) and a React UI layered on GPUIX (React over GPUI-style native rendering). Design chat builds directly on the canvas with agent cursors, highlight flashes and live streaming; 1–4 parallel agents split work or make variants; "Let it cook" continues variations autonomously; backends are your Claude Code login or a Claude API key; an MCP server on 127.0.0.1:47700 exposes the same tools to Claude Code/Cursor/Codex. Pen tool for vectors; Figma import and SVG I/O; HTML/script frames.

## Facts
| | |
|---|---|
| GitHub | https://github.com/naddrzz/nalarupa |
| License | MIT (LICENSE) |
| Stack | Rust core (22 files, 20.3k LOC), React + GPUIX (@gpuix/native, @gpuix/react), MCP over Streamable HTTP, Claude backends |
| Size | app/ 18 TS files (6.3k) + core/ 22 rust files (20.3k); assets/templates/guide in core/ |
| Activity | no git history in download |
| Tests | ~190 `#[test]` fns inside core/src files; 0 app-side tests; 7 CI workflows incl. claude-ci-fix.yml, claude-qa.yml, heartbeat.yml |
| Entry points | `app/src/main.ts` (desktop), `core/src/bin/cli.rs` (headless), `core/src/mcp.rs` (MCP server) |

## Architecture
- **app/src** — Canvas.tsx (7 tools, camera, snapping, guides), Agent.tsx panel (chat/plan/queue/permission card/model picker), AgentLayer (cursors/glow), Inspector/Layers, Present/Home; core.ts is the call()/renderPixels() bridge. [overview](scan-overview.html)
- **core/src** — engine.rs (tools/undo), doc.rs (document model + agent commit batching), agent.rs (1.6k built-in loop), claude.rs (two backends), mcp.rs (HTTP MCP), render.rs/raster.rs/shader.rs (GPU pipeline), figma.rs/svg.rs (import/export), html.rs/script.rs (canvas frames), pen.rs, net.rs, resolve.rs, guide.rs.
- **core/tests** — figma-clipboard fixture.

## Quality evidence
- Builds / type-checks: not run (cargo/bun fetch not permitted).
- Tests: ⚠️ ~190 rust test fns in src (no separate tests dir), 0 renderer tests; CI includes agent-driven fix/qa loops (claude-ci-fix.yml).
- Structure: ✅ crisp two-layer split (rust engine / thin React app); pen and geometry math isolated as pure TS modules.
- Error handling: permission card before tools run outside the canvas; undo batching for agent edits.
- Docs: ✅ README features (cursors, parallel agents, let-it-cook, model picker, MCP) all map to code; guide/ + templates/ ship with the core.
- Dependencies: @gpuix runtime packages; anthropic-compatible backends.
- Maintenance: not checked (no git history); agent-automation CI (heartbeat, retry-claude) suggests active development.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired*
- [ ] Doesn't build — *not fired* (not verified)
- [x] No renderer-side tests — the React layer is untested; rust core carries the assertions

## Verdict rubric
**good** — clean architecture, thoughtful agent UX (undo batching, permission card, shared MCP), honest docs; 2 slop signals (no history, thin renderer tests).

## Worth stealing
- **Agent edits as one undo step**: stream live, commit atomically (`core/src/doc.rs`).
- **Agent cursors as a protocol**: named/colored cursors + flashes + generating glow driven by engine events (`AgentLayer.tsx`).
- **One MCP surface for internal and external agents** (`core/src/mcp.rs`) — no second-class tools.
- **GPUIX pattern**: React ergonomics over a rust/GPU render core with pixel readback.
- **Permission card before out-of-canvas tool runs.**

## Structural findings
1. **engine.rs/doc.rs own everything mutable** — clean funnel; UI never touches the doc directly. (info)
2. **Renderer untested** — all 190 tests are rust-side; the interaction layer (snapping, pen) is pure TS and testable but isn't. (warn)
3. **Figma import in core** — figma.rs (1.5k) + svg.rs (2.2k) give real interop from day one. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
