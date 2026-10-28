---
slug: swarajbachu__zuse
name: zuse
github: https://github.com/swarajbachu/zuse
license: AGPL-3.0-only
scanned: 2026-10-27
verdict: good
stack: [typescript, effect, react, electron, bun, turbo, vitest, acp, sqlite]
---

# zuse

> Chat-first agent desktop: Electron shell, Effect-TS agent kernel, ACP.

**Verdict: good** — 1,176 test files, an architecture-boundary checker in CI, Effect-Schema contracts shared across renderer/server/agents, and ACP as a first-class driver; the cost is a 158k-LOC renderer with one flat 200-file components dir.

## What it is
Zuse (alpha, v0.22.1) is a chat-first desktop app for developers working with AI coding agents — Claude Code, Codex, Grok, Gemini, Cursor, OpenCode — wrapped in a persistent, project-aware interface: structured chat history, rich composer, file viewer, integrated terminal, git worktrees, all stored locally (packages/sqlite). Built as a Bun + Turborepo monorepo: an Electron desktop shell (apps/desktop), a big React renderer (apps/renderer, 158k LOC), a Node server (apps/server, 87k), an Effect-TS agent kernel with 54 provider drivers (packages/agents) speaking ACP JSON-RPC (packages/acp), plus mobile, web, slack and an MCP server.

## Facts
| | |
|---|---|
| GitHub | https://github.com/swarajbachu/zuse (README links) |
| License | AGPL-3.0-only (LICENSE GPL-3 text; package.json `license: "AGPL-3.0-only"`; paid icons gated by scripts/install-paid-icons.mjs) |
| Stack | TypeScript, Effect (Schema/layers), React 18, Electron, Bun + Turborepo, Biome, Vitest + Playwright, sqlite, ACP JSON-RPC |
| Size | renderer 158.9k LOC, server 87.7k, mobile 37.5k, web 20.5k, desktop 15.9k, slack 5.8k; packages: agents 34.3k, contracts 18.2k, client-runtime 14.2k, domain 9.7k, git 9.1k |
| Activity | no git history in download |
| Tests | 1,176 `*.test.*` files; turbo test layers (unit/integration/system/desktop) + windows-tests workflow; check-architecture-boundaries script |
| Entry points | `apps/renderer/src/main.tsx` (Electron webview), `apps/desktop/src` (main process), `apps/server/src/bin.ts`, `packages/agents/src/kernel` |

## Architecture
- **apps/renderer** — the workspace UI: chat timeline, rich composer, bottom terminal dock, browser panes; hand-rolled atom store (state/atom-store.ts) with 20+ per-concern stores (chats, acp-agents, composer-drafts, git-stack, masters). [overview](scan-overview.html) [renderer map](scan-renderer.html)
- **packages/agents** — the kernel: kernel/ (turn protocol, permission policy, ACP session, provider process supervision), 54 driver files (claude, codex, gemini, cursor, opencode, acp...), harness/, mcp-gateway (browser tools served to agents). [agents map](scan-agents.html)
- **packages/acp** — ACP JSON-RPC schemas + connection/rpc-client; any ACP-capable agent works through one adapter.
- **packages/contracts** — Effect Schema contracts shared by renderer, server and agents (one source of runtime+static types).
- **apps/server** — backend API: auth, checkpoints, collaboration, automation, attachments; relays chats to mobile/Slack/web.
- **apps/desktop** — Electron main process: windows/tray/awake/auth callbacks/deep links; renderer talks over the IPC bridge.
- **packages/sqlite + git + client-runtime + serve** — local persistence, git/worktree services, typed API client, headless CLI servers.

## Quality evidence
- Builds / type-checks: not run (bun install needs network; `check-types` script exists and includes the boundary checker).
- Tests: ✅ 1,176 test files; turbo test:unit/integration/system/desktop; windows-compat CI workflow; architecture + API-terminology checkers.
- Structure: ⚠️ renderer is one 158k app with a flat ~200-file components dir; packages stay clean (agents 34k with tight kernel/driver split); boundary checker compensates.
- Error handling: ✅ 0 `@ts-ignore`; 20 TODO/FIXME mentions across ~350k TS LOC; Effect typed errors in contracts.
- Docs: ✅ README matches code (supported agents ↔ drivers/, worktrees ↔ packages/git, terminal ↔ renderer terminal dock); AGENTS.md + CONTEXT-MAP.md + DESIGN.md + specs/ present.
- Dependencies: Bun catalog + patches/ (pinned forks); paid icon assets gated at postinstall with a status checker.
- Maintenance: not checked (no git history in download); CHANGELOG present.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired* (vendor/runtime.mjs is a declared vendored engine)
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (THIRD_PARTY_NOTICES.md)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real tests, contract-driven layering with an automated boundary check, honest alpha docs; 1 slop signal.

## Worth stealing
- **ACP as a driver**: one JSON-RPC adapter (packages/acp + kernel/acp-session) makes any ACP agent a first-class citizen next to bespoke CLI drivers.
- **Contracts-as-schemas**: Effect Schema types shared across renderer/server/agents keep protocol drift visible at compile time.
- **Architecture boundary checker**: scripts/check-architecture-boundaries(.test.mjs) enforces the layering in CI — cheap and rare.
- **Permission policy as a module**: permission-policy.ts centralizes tool-approval semantics the UI only renders.
- **Local-first desktop + relay server**: desktop owns sqlite truth; server relays to mobile/slack without becoming the store.

## Structural findings
1. **renderer is the monolith** — 158k LOC, ~200 flat component files; the boundary checker holds the line but domain grouping (chat/browser/terminal) is the next lever. (warn)
2. **kernel/driver split is exemplary** — provider quirks stay in 54 driver files; turn-protocol/permissions/stream-text are shared and testable. (info)
3. **server as relay, not store** — desktop keeps local-first truth; mobile/Slack/web subscribe through apps/server. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| agents | [scan-agents.json](scan-agents.json) | [scan-agents.html](scan-agents.html) |
| renderer | [scan-renderer.json](scan-renderer.json) | [scan-renderer.html](scan-renderer.html) |
