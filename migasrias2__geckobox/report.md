---
slug: migasrias2__geckobox
name: geckobox
github: https://github.com/migasrias2/geckobox
license: MIT
scanned: 2026-10-27
verdict: good
stack: [typescript, node, react, vite, zod, vitest, ssh, docker, vnc]
---

# geckobox

> CLI + local web UI to run coding agents on any computer, with no relay.

**Verdict: good** — 12 focused vitest files matching the source layout, exactly one runtime dependency (zod), secrets kept out of argv where transports allow, and a README that documents its one honest trade-off; small enough (~6k LOC) that quality is uniform.

## What it is
Geckobox ships as `openspaces`: a Node 20+ CLI and SDK (`src/`) plus a local dashboard (`ui/`, React 19 + Vite). You register computers (local, `ssh:user@host`, `container|docker|podman:name`, `cua:name`), then start agent threads — Claude Code, Codex, Gemini CLI, etc. — that run inside work directories on those machines via shell scripts piped over `sh -s`. Turn scripts write a pid file, the agent streams JSONL events which get folded into thread state, and jobs can fan out to many computers at once (`run --on a,b,c`). A 127.0.0.1-only web UI serves the live screen panes and chat. No relay: everything connects directly, so there is nothing to meter (the README's framing against Cua).

## Facts
| | |
|---|---|
| GitHub | https://github.com/migasrias2/geckobox (`npm install -g github:migasrias2/geckobox`) |
| License | MIT (LICENSE + `package.json` `license: "MIT"`; © 2026 Miguel Fortes Martins) |
| Stack | TypeScript 5.9, Node ≥20 ESM, zod ^4, React 19, Vite 8, Tailwind 4, vitest 4; no runtime deps beyond zod |
| Size | 58 TS files, ~6,019 LOC (src ~1.7k core + server/agents/cli ~1.25k + ui/); no git history in download |
| Activity | no git history in download |
| Tests | 12 vitest files in `test/` mirroring src modules (agents, computers, thread, turn, transport, server, screen, shell, sandbox, view, cli); no CI workflows in this drop |
| Entry points | `src/cli.ts` (bin `openspaces`), `ui/src/main.tsx` (dashboard), `src/index.ts` (SDK exports) |

## Architecture
- **src/cli.ts** — parseArgs dispatch; HELP is the manual. Commands live in `cli/commands.ts` (start/send/wait/status/logs/interrupt/close/run) and `cli/sandbox.ts`. [overview](scan-overview.html)
- **src/thread.ts → src/turn.ts → src/transport.ts** — the funnel: turn options validated with zod, a turn script is built pure (pid-file, workdir, missing-binary check), then `invocationFor()` wraps it per computer kind (script always on stdin except the documented Cua caveat).
- **src/store.ts** — thread state + append-only event log on disk; src/worker.ts detaches `cli.js __worker` so turns outlive the caller.
- **src/agents/** — per-binary adapters (argv construction + JSONL event parsing; `claudeCode.ts` also extracts Cua tool targets/points for screen annotation).
- **src/server/** — 127.0.0.1-only http server with zod-validated `/api` routes (`routes.ts`), hashed-secret-token link (`auth.ts`), the built UI (`static.ts`), screen frames (`screenHub.ts`) and a transcribe route for dictation.
- **ui/src** — React app: Sidebar/NewThread/ThreadView/Composer, LiveScreen polling `/api/screen` by seq, Computers/Sandboxes management; tiny homegrown hooks (`useThread` 900ms incremental fetch, `usePoll`), no state library.
- **Desktop viewing** — `view.ts` computes a plan: SSH tunnel to VNC 5900 and open the local port, self-view, or Cua's viewer; no relay involved.

## Quality evidence
- Builds / type-checks: not run (npm install not permitted in this pass); both `npm run typecheck` and `ui` typecheck scripts exist and `tsc --noEmit` is part of the UI build script.
- Tests: ✅ 12 test files / `test/`, one per src module pattern; `vitest run` + coverage script present.
- Structure: ✅ no file over ~243 LOC (largest: `src/server/routes.ts` 243); module-per-concern with few public exports.
- Error handling: ✅ 2 TODO/ts-ignore matches total; explicit id regex guards (`SAFE_ID` in turn.ts, port range check in view.ts); env-name and tool-pattern zod regexes at every boundary.
- Docs: ✅ README claims match code — sandbox (~20s create) ↔ `sandbox.ts`, secrets in Keychain ↔ `secrets.ts`, "no relay" ↔ direct ssh/tunnel paths, Cua argv caveat documented in `transport.ts` comments and README Safety section.
- Dependencies: ✅ exactly one runtime dep (zod ^4.1.0); dev deps typescript/vitest only; UI adds react 19 + vite 8.
- Maintenance: not checked (no git history in download); `docs/` carries a tutorial with screenshots.

## Slop signals
- [ ] README promises features that have no code — *not fired*: each quick-tour feature maps to a module above
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired*: clean-room claim documented; MIT license intact
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — builds plausibly, tight module structure, real per-module tests, one-dependency budget, 1 benign slop signal.

## Worth stealing
- **Transport as a tiny seam**: `invocationFor(computer, script, env)` turns any computer kind into `sh -ls` with the script (and secrets) on stdin — `src/transport.ts` is 84 lines.
- **Pure turn building/folding**: `turnScript()` and `foldEvents()` are pure functions, which is why test-turn.test.ts can cover correctness without process spawning.
- **Reuse of the CLI as a worker**: the UI server doesn't own process management; it detaches `cli.js __worker` so CLI and UI share one lifecycle (`src/worker.ts`).
- **Token-in-hash auth for localhost tools**: single-use secret link from `auth.ts` with hashchange pickup in `ui/src/App.tsx`.
- **Screen hub with idle stop**: `screenHub.ts` throttles capture per computer, long-polls by seq, and stops when nobody looks.

## Structural findings
1. **transport.ts is the whole boundary** — every computer kind rides one `sh -s` script seam; adding a kind is one case in `invocationFor`. (info)
2. **routes.ts at 243 LOC is already the biggest file** — clean but on the path to becoming a table overflow point if more endpoints arrive. (info)
3. **no background long jobs in the UI server** — worker processes and screen capture loops are detached/hubbed, so UI restarts don't strand threads; only pid files ground truth on computers. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
