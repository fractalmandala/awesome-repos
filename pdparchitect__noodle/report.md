---
slug: pdparchitect__noodle
name: noodle
github: https://github.com/pdparchitect/noodle
license: Apache-2.0
scanned: 2026-10-27
verdict: good
stack: [swift, tuist, appkit, swiftui, metal, wkwebview, mcp]
---

# noodle

> Swift/Tuist suite: Noodle agent teams + Computer, Browser, Applet, Hub, Mobile.

**Verdict: good** — 453 test files (~2,087 test funcs, 71k test LOC vs 124k app LOC), 12 release workflows, and a disciplined per-provider harness pattern; the suite is large but cleanly separated.

## What it is
Noodle is a macOS 26+ native suite (Tuist-generated Xcode projects, Swift/SwiftUI/AppKit) for running AI agent "teams": give a bot a task or group several toward a shared goal; each bot has its own workspace and backstory and runs on the harness you already pay for (Codex, Claude Code, FX, Grok Build, Muse Code, OpenCode v2, Antigravity, Apple Intelligence on-device). Companion apps: Noodle Computer (Linux/macOS guest computers with a Metal kernel), Noodle Browser (persistent-profile WKWebView browsers assigned to agents), Noodle Applet (runs tools/sites/games agents build), Noodle Hub (share harnesses to family devices), and Noodle Mobile (TestFlight iOS client for the Hub).

## Facts
| | |
|---|---|
| GitHub | https://github.com/pdparchitect/noodle |
| License | Apache-2.0 (LICENSE + NOTICE) |
| Stack | Swift, SwiftUI/AppKit, Tuist (Project.swift per app), Metal (Computer Kernel), WKWebView, MCP; signed connection bridges |
| Size | 650 non-test swift files, 123.6k LOC: Sources 62.8k, Computer 25.5k, Shared 18.7k, Mobile 13.4k, Applet 13k, Browser 6.9k, Hub 1.9k |
| Activity | no git history in download; VERSION file + CHANGELOG present |
| Tests | 453 test files / ~71k LOC / ~2,087 `func test`; CI: 12 workflows incl. per-app release pipelines + testflight + website |
| Entry points | `Sources/Noodle/NoodleApp.swift` (@main), per-app Tuist projects, `Hub`, `Computer/Kernel/build.sh` |

## Architecture
- **Sources/Noodle** (92 files) — main app UI over one NoodleStore: agent pickers/profile sheets, chat surfaces, annotations, browser tabs. [overview](scan-overview.html) [app map](scan-noodle-app.html)
- **Sources/NoodleCore** (~150 files, 45k) — the domain core; every harness gets a file-triple of Distribution / ExecutableTrust / Launch / SessionState / Inspection (8 providers), plus WorkspaceRepository, AgentStorageLayout, UsageLedger, DeletionJournal, tips, trust checks.
- **Sources/NoodleAgentHost + Bridge** — sandboxed out-of-process host running agent CLIs; trust-verified before every spawn; turns stream back over the bridge.
- **Computer/** — guest computers: Kernel (Metal display), Bridge (signed connection), LocalMac, guest images; agents can run remotely with file transfer.
- **Browser/ + Applet/ + Hub/ + Mobile/** — companion apps sharing the Shared package; Hub serves harnesses to Mobile over the bridge.
- **Shared/** (18.7k) — cross-suite Swift package (models, UI atoms, transport).

## Quality evidence
- Builds / type-checks: not run (Tuist/Xcode generation needs local tooling; skipped, recorded).
- Tests: ✅ 453 test files (HubCoreTests, NoodleAppTests, AppletLinkIntegrationTest, ...); 12 CI workflows.
- Structure: ✅ clear seams: per-app dirs + shared package + domain core; one store in the main app; host process isolation.
- Error handling: ✅ 0 TODO matches in Sources/Shared; trust/signature-verification files for every provider binary path.
- Docs: ✅ docs/ covers harness-setup, mcp-connections, keyboard-shortcuts, security, privacy; README claims (group goals, hub sharing, applets) all map to code.
- Dependencies: Apple frameworks only + vendored packages via Tuist; no third-party SDK sprawl.
- Maintenance: not checked (no git history); 12 release workflows imply disciplined shipping.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired* (Tuist manifests are small)
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [ ] Copied upstream without attribution — *not fired* (NOTICE present)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — deep test coverage, honest docs, strong separation, 1 slop signal.

## Worth stealing
- **Provider file-triple**: Distribution/ExecutableTrust/Launch/SessionState per harness isolates CLI quirks and makes trust checks uniform (`Sources/NoodleCore/Claude*`, `Codex*`, ...).
- **Agent host out-of-process**: CLIs run sandboxed behind a bridge; UI never spawns providers directly (`Sources/NoodleAgentHost`).
- **DeletionJournal**: safe-delete journal for agent-owned files — an undo path for destructive agents.
- **Device-local viewport state**: tabs/viewport stay local while data lives in core — same idea as zeron's rail.
- **Suite release pipelines**: independent per-app release workflows from one Tuist monorepo.

## Structural findings
1. **NoodleStore fan-in** — 92 view files observe one central store; fine now, watch as the app grows. (warn)
2. **Computer is a real second app** — Kernel/Bridge/LocalMac with its own Metal renderer and signed bridge; biggest independent surface after the main app. (info)
3. **test-first culture** — 71k test LOC vs 124k app LOC, integration tests wired into CI per app. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
| noodle-app | [scan-noodle-app.json](scan-noodle-app.json) | [scan-noodle-app.html](scan-noodle-app.html) |
