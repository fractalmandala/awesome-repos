---
slug: fei2411__deepseek-bot
name: DS Bot
github: https://github.com/Fei2411/DeepSeek-Bot
license: Apache-2.0
scanned: 2026-10-27
verdict: good
stack: [javascript, node, electron, react, deepseek-harness, playwright, docker]
---

# DS Bot

> A DSH plugin that turns DeepSeek Harness into a team of long-lived Bots, plus a desktop app.

**Verdict: good** — a deep, disciplined plugin and shell: 35 host modules wired by one ordered list, 110 unit files plus a scripted mock model, a leak scan in CI, and an Electron shell that only talks to its own loopback Host.

## What it is
DS Bot makes DeepSeek Harness (DSH) behave as a team of Bots. It is a DSH plugin (`ds-bot`) with a server side that registers tools (`create_bot`, `message_bot`, `create_group`, `remember`, `read_browser`…), and a React client that injects itself into DSH's UI slots as a team surface. A Main Bot ("Chief") manages the others; Bots message each other, hold group chats with ordered rounds, keep Markdown memory, drive a per-Bot browser, and hand work off. `app/` is an Electron desktop app that ships its own pinned DSH, boots a private profile on loopback, and shows DSH Web in its window.

## Facts
| | |
|---|---|
| GitHub | https://github.com/Fei2411/DeepSeek-Bot (from every package manifest; README install links use the `DS-Bot` alias) |
| License | Apache-2.0 (`LICENSE`, every `package.json`), with third-party notices in `NOTICE` |
| Stack | Node ≥ 22.18 (CI 22/24), ESM JavaScript, React 19.3 (client), DSH 0.2.1-rc.2 (peer + pinned), Electron 44.7 + electron-builder 26.17 + electron-updater, Playwright 1.64, oxlint 1.76, esbuild 0.28, `ws`, Vite 8.3 (parked window), Docker |
| Size | 466 files; `ds-bot/src` 34,443 LOC (host 16,026 in 52 files, browser 1,419, client 17,398 in 74, shared 1,019); `ds-bot/test` 24,259 in 110 files; `app/src` 3,749 in 22; `app/ui` 795 (parked); docs 21 files / 2,175 lines; generated `client.js` 1.1 MB |
| Activity | no git history in download (folder `DS-Bot-main`); v0.2.2 dated 2026-10-25 in `CHANGELOG.md` with PR links |
| Tests | 110 `ds-bot` test files, 17 app tests, 2 browser-plugin tests; a scripted mock model (`dev/mock-llm.mjs`), a smoke bench (`dev/bench.sh`, `dev/smoke.mjs`), stale-bundle check (`build-client.mjs --check`), leak scan (`scripts/scan.mjs`) |
| Entry points | `ds-bot/index.js` (plugin), `ds-bot/src/client/index.js` (client bundle source), `app/src/main.mjs` (Electron), `docker/entrypoint.sh` |

## Architecture
- **Plugin host** (`ds-bot/src/host/`): 35 modules installed in a fixed order from `index.js`; they share one runtime object `rt` and one `state.json` team record (roster, rooms, agents, exchanges, prefs, revision).
- **Client** (`ds-bot/src/client/`): React views that shadow 30+ DSH slots (`sidebar.*`, `conversation.chat.node`, `conversation.input.dock`, `shell.overlay`…), with hand-rolled `createSource` stores and a Bot-mode/Agent-mode surface switch.
- **Transport**: `POST /api/bot` (~40 endpoints), an SSE feed for live turns, and a long-poll path for the Bot browser.
- **Long-lived conversations**: `parts.js` + `steward.js` decide between condensing in place and moving to a fresh Session with a handoff note; memory and chats are indexed for `read_own_chat`.
- **Desktop app** (`app/`): spawns `host.mjs` under Electron's Node mode, which boots a pinned profile (`dsh-base` + `dsh-web-app` + `ds-bot`) on a loopback port; the window loads that URL, and a narrow preload bridge leases Bot webviews.
- **Browser plugins** (`browser-plugins/`): three interchangeable Playwright backends (refs, script, task), one per team, one Chromium profile per Bot.

Maps: [overview](scan-overview.html) · [host](scan-host.html) · [client](scan-client.html) · [app](scan-app.html).

## Quality evidence
- Builds / type-checks: ⚠️ `npm test`, the bundle check and lint were **not run** — the repo has no `node_modules` and `npm run harness` installs `@deepseek-ai/*` from npm. As a smoke test I ran `node --check` on all 169 `.js`/`.mjs` files in `ds-bot/src`, `app/src`, `app/ui/src` and `browser-plugins`: 0 syntax errors.
- Tests: ✅ 110 unit files with a mock Anthropic-compatible model that drives whole team flows (set up, create_bot, relays, group rounds, questions); CI runs them on Linux/Windows, Node 22/24 and DSH 0.2.1-rc.2/0.2.0-rc.2, plus `build-client.mjs --check` and `oxlint`.
- Structure: ⚠️ consistent `install(rt)` modules and clear separation host/client/app, but `host/memory.js` (1,748), `host/parts.js` (1,325) and `host/groups.js` (1,000) mix state, tools and I/O; the client's `styles.js` (1,217), `i18n.js` (1,160), `cells.js` (1,024) and `index.js` (922) are 4.3k of 17.4k.
- Error handling: ✅ zero `TODO`/`FIXME`/`HACK` in shipped code, one lint suppression, typed refusals (`bot/refused`, `bot/invalid`, `bot/internal`) so programming errors are logged but user refusals are not; a diagnostics ring feeds Settings → Feedback.
- Docs: ✅ `README.md` and `README.zh.md` kept in step, 15 design notes in `docs/plans/`, `AGENTS.md` + `CONTRIBUTING.md` with hard rules and a PR workflow. Drift: the install block still names `ds-bot-0.2.1.tgz` and the badge says `v0.2_preview` while `package.json` is 0.2.2.
- Dependencies: ✅ exact pins everywhere, DSH pinned to one rc, `overrides` absent, release publishes with npm Trusted Publishing (OIDC, provenance) and no stored token. No Dependabot/Renovate config was found.
- Maintenance: ✅ CHANGELOG per release with issue/PR links, release workflow builds Windows/macOS/Linux artifacts, a "leak scan" job checks the archived tree for secrets. Activity itself cannot be judged without `.git`.

## Slop signals
- [ ] README promises features that have no code
- [ ] Large generated-looking files
- [ ] Dead code / placeholder impls
- [ ] Inconsistent architecture
- [ ] One or two giant commits containing the whole project
- [ ] Copied upstream without attribution
- [ ] Doesn't build / obvious compile errors
- [ ] Tests that assert nothing

None ticked. Two notes that are not slop: `ds-bot/client.js` is a committed build with a CI staleness check, and `app/ui` + `app/engine` are parked by design and documented as on hold.

## Verdict rubric
- **good**: clear structure, deep tests, license + notices, CI that checks more than tests, ≤ 1 slop signal. ✅

## Worth stealing
- `ds-bot/dev/mock-llm.mjs` — a scripted Anthropic-compatible model that drives full multi-agent flows, so team behaviour is testable without a key or a network.
- `AGENTS.md` + `dev/check.sh` — the "done" definition (tests, bundle check, lint) written once and enforced in CI.
- `ds-bot/src/host/home-lock.js` — a one-writer lock on the plugin home so two DSH hosts cannot clobber `state.json`.
- `ds-bot/src/host/approvals.js` — answering DSH approvals for hidden member Sessions, which is what keeps a group round from hanging.
- `scripts/scan.mjs` — a leak scan over `git archive HEAD`, with a separate `SCAN_EXTRA_PATTERNS` channel for machine- or account-specific markers.
- `app/src/cookie-import.mjs` — parsed cookies stay in main under a pending id; the page only ever sees a value-free preview.

## Structural findings
- **Registration order is load-bearing** — `ds-bot/index.js:77` lists all 35 host modules; listeners and tools run in that order, so `usage` must stay last to meter every rewritten request. A new module must be placed, not appended.
- **Three host modules hold half the logic** — `memory.js` 1,748, `parts.js` 1,325 and `groups.js` 1,000 are 4.1k of the host's 16k lines, each mixing state, tools and I/O.
- **The client is a slot tenant** — `ds-bot/src/client/index.js:793-851` shadows 30+ DSH slots and withdraws them all in Agent mode, so the UI lives inside DSH rather than beside it.

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| host | [scan-host.json](scan-host.json) | [scan-host.html](scan-host.html) |
| client | [scan-client.json](scan-client.json) | [scan-client.html](scan-client.html) |
| app | [scan-app.json](scan-app.json) | [scan-app.html](scan-app.html) |
