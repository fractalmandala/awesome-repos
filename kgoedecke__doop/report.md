---
slug: kgoedecke__doop
name: doop
github: https://github.com/kgoedecke/doop
license: AGPL-3.0-only
scanned: 2026-10-27
verdict: good
stack: [typescript, react, express, ws, drizzle, postgresql, better-auth, mcp, tauri, swift]
---

# doop

> Multiplayer design canvas where humans and AI agents co-design over live HTML frames.

**Verdict: good** — 116 test files (16k test LOC), a single shared mutation funnel (server/actions.ts) consumed by REST, ws, MCP and the resident agent loop, zero TODO/`@ts-ignore` smell, and real CI/release automation; the weak spots are two ~2k-LOC files.

## What it is
doop is a self-hostable web app: an infinite multiplayer canvas whose frames are live HTML documents. Humans draw and edit; AI agents either connect over MCP from an external client (Claude Code, ChatGPT), run as a built-in server-side "resident design team" (per-role Claude tool loops over Anthropic/OpenAI/OpenRouter/Gemini/user keys), or run locally through the desktop shell's installed Claude CLI. Distributed as a Bun/Express server + Vite React client (`bun run dev`), plus a Tauri desktop shell, a ChatGPT plugin listing, and an iPhone/iPad app with Live Activities.

## Facts
| | |
|---|---|
| GitHub | https://github.com/kgoedecke/doop |
| License | AGPL-3.0-only (LICENSE first line "GNU AFFERO GENERAL PUBLIC LICENSE"; `package.json` `license` field agrees) |
| Stack | TypeScript 5.5, React 18, Vite 8, Tailwind 4, express 4, ws 8, drizzle-orm 0.45 + pg, better-auth 1.6.26, @anthropic-ai/sdk 0.115, @modelcontextprotocol/sdk 1.12, stripe 21, puppeteer-core 25, zustand 4; desktop Tauri 2 (Rust 413 LOC), iOS Swift |
| Size | 238 TS files, ~52.5k LOC core (src 25.1k, server 25.7k, shared 1.6k) + 111 test files in tests/ (16.1k LOC); not counted: desktop Rust 413, ios Swift |
| Activity | no git history in download (zip); CHANGELOG has releases up to v0.8.0 (2026-10-13) |
| Tests | 111 vitest test files (`tests/*.test.ts(x)`), 16,118 LOC; CI in `.github/workflows/ci.yml` plus release-please, iOS, desktop and CLA workflows |
| Entry points | `server/index.ts` (server), `src/main.tsx` → `src/App.tsx` (web), `desktop/src-tauri/src/main.rs` (desktop shell), `chatgpt-plugin/plugin.json` (ChatGPT listing) |

## Architecture
- **server/index.ts** — monolithic entry: ~60 inline REST routes, the `/ws` multiplayer room, MCP/Gemini-cloud endpoints, static web serving, 9 mounted routers (`server/admin.ts`, `billing.ts`, `automations.ts`, `workspaces.ts`, `community.ts`, `integrations.ts`, `notifications.ts`, `webhookRoutes.ts`, `localAgent.ts`). [overview](scan-overview.html)
- **server/actions.ts** (1,679 LOC) — every mutation: frames, tasks, chat, comments, memory, activity log, broadcast and persist. REST, ws handlers, MCP tools, the resident loop and the Linear/Slack/GitHub extensions all funnel through it.
- **server/store.ts + server/db/persist.ts + db/schema.ts** — in-memory canvas Map hydrated from Postgres at boot, with write-through mirroring of every mutation; orphaned frames self-heal into page 1 at hydrate (`store.ts init()`).
- **server/mcp.ts** (1,757 LOC) — the agent surface: ~20 tools (create_frame, streamed append_frame_html, edit_frame_html, get_frame_screenshot, feedback/comments, guidelines, memory, image search/generation, webpage import) with a long built-in workflow prompt.
- **server/resident.ts** (1,574 LOC) — built-in agent team; per-role Claude tool loops that use the same actions, with model-tier fallback from server keys onto the human's connected plan (server/agentModel.ts).
- **src/main.tsx → src/App.tsx** — hand-rolled popState router dispatching 10 route patterns; all canvas wiring lives in **src/pages/CanvasPage.tsx** (1,672 LOC). [canvas-web map](scan-canvas-web.html)
- **src/lib/store.ts** — single zustand store (canvas, presence, cursors, tasks, chat, comments, memory proposals) driven by **src/lib/ws.ts**, which reconnect-codes 4401/4403/4404.
- **desktop/src-tauri/src/main.rs + claude.rs** — thin Tauri shell over the hosted web app, exposing `claude_status`/`claude_connect` bridge commands; **src/lib/localAgent.ts** reads `window.__TAURI__` and degrades to plain browser.
- **ios/** — native Swift app: agent status + LiveActivities (APNs push via `server/apns.ts`, `server/liveActivities.ts`).
- **shared/** — types/helpers (agents, pages, webhooks, model menus, automations) duplicated for both client and server. [server map](scan-server.html)

## Quality evidence
- Builds / type-checks: not run — no network installs allowed in this pass; `tsx`/`bun` scripts exist (`npm run typecheck` via tsgo).
- Tests: ✅ 111 test files / 16,118 LOC in `tests/`, run by vitest (`package.json test` script); CI runs it (`.github/workflows/ci.yml`).
- Structure: ⚠️ clean module boundaries (actions funnel, routers, extensions) but two monoliths: `server/index.ts` 1,991 LOC and `src/pages/CanvasPage.tsx` 1,672 LOC; `server/mcp.ts` 1,757 and `server/resident.ts` 1,574 close behind.
- Error handling: ✅ 0 `TODO|FIXME|HACK` and 0 `@ts-ignore|@ts-expect-error` matches in src/server/shared; `unwrap(`/`expect(` absent (JS codebase); ws close-code routing is unusually careful (`src/lib/ws.ts`).
- Docs: ✅ README (45.8k) matches code — pages feature ↔ `src/components/PagesPanel.tsx` + `shared/pages.ts` + `server/index.ts` `/api/canvases/:id/pages`; built-in agent ↔ `server/resident.ts`; MCP setup ↔ `server/mcp.ts` + `src/components/ConnectModal.tsx`; docker compose ↔ `Dockerfile`/`docker-compose.yml`. No promised feature found without code.
- Dependencies: ~40 runtime deps, versions loose-caret pinned via bun.lock; better-auth/jose/nanoid exact-pinned. Stripe, drizzle, MCP SDK all current majors.
- Maintenance: not checked (no git history in download); release-please + CHANGELOG through 2026-10-13 suggests active cadence.

## Slop signals
- [ ] README promises features that have no code — *not fired*: every README feature checked had a real implementation path
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*: 0 `todo!`/`unimplemented!`/`// TODO` matches in src/server/shared
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — the zip has no `.git`; iteration history unverifiable (only signal is the CHANGELOG)
- [ ] Copied upstream without attribution — *not fired*; AGPL license file intact
- [ ] Doesn't build — *not fired* (not verified either)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, precise architecture, up-to-date tooling, honest README, 1 benign slop signal.

## Worth stealing
- **Single mutation funnel**: server/actions.ts is the only writer of canvas state; every surface (REST, ws, MCP, resident loops, extensions) reuses it, so activity/broadcast/persist invariants can't be drifted.
- **Hydrate self-healing**: server/store.ts re-attaches frames with dead pageIds into page 1 during boot hydrate — cheap crash-window repair.
- **Close-code routing on reconnect**: src/lib/ws.ts maps 4401/4403/4404 to distinct recovery behaviors instead of a blind retry loop.
- **Prompt-as-data contracts**: shared/localAgent.ts and shared/modelMenu.ts keep agent/model catalogs as shared data both shells read.
- **CLI preference owned by server**: src/lib/localAgent.ts saves the local-Claude preference server-side and treats server routing as authoritative, so the CLI state can't disagree.

## Structural findings
1. **CanvasPage is a god-layout** — src/pages/CanvasPage.tsx (1,672 LOC) renders ~15 components directly and owns import modals, shortcuts, share, present and onboarding wiring. (alert)
2. **index.ts is a monolith** — server/index.ts (1,991 LOC) inlines ~60 REST routes, the ws room and boot logic instead of splitting routers. (warn)
3. **memory + Postgres dual sources** — state lives in server/store.ts Maps; db/persist.ts write-through mirrors each change; hydrate patches crash windows. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| canvas-web (src/) | [scan-canvas-web.json](scan-canvas-web.json) | [scan-canvas-web.html](scan-canvas-web.html) |
| server | [scan-server.json](scan-server.json) | [scan-server.html](scan-server.html) |
