---
slug: mariomile__kore
name: kore
github: https://github.com/mariomile/Kore
license: unknown
scanned: 2026-10-27
verdict: good
stack: [typescript, react, tauri, rust, sqlite, mcp, icloud]
---

# kore

> Reflect fork: plain-file notes with tabs, graph, agents, Automations.

**Verdict: good** — 384 colocated `*.test.*` files, 7 CI workflows, a written design system, and a privacy-strong agent design (private notes blocked at CLI sandbox level); upstream is MIT but this fork ships no LICENSE file.

## What it is
Kore is a personal fork of Reflect (team-reflect/reflect-open, MIT, local-first Markdown notes) that adds a Craft/Linear-grade design language (ink hairlines, one accent, raised pills; Space/Midnight/Paper themes), Linear-style note tabs (⌃Tab cycle, ⌘W close, double-click pin, persisted per graph), dual collapsible rails, a force-directed graph view, an in-app sandboxed browser, file viewers (PDF/HTML/DOCX/CSV), and subscription AI through the locally installed Claude Code or Codex CLI — including Sign in with ChatGPT — with `private: true` notes hard-blocked at the CLI sandbox level. Agents have identity ("soul") and memory files, shared signed-facts memory, write approval, and scheduled Automations; MCP servers configured in-app with tokens in the OS keychain.

## Facts
| | |
|---|---|
| GitHub | https://github.com/mariomile/Kore (README self-reference; upstream team-reflect/reflect-open) |
| License | unknown — fork of MIT reflect-open but no LICENSE file in this drop (flagged) |
| Stack | TypeScript + React 19, Tauri 2 (desktop + iOS), Rust core (30.1k), SQLite, design-system tokens, MCP |
| Size | 1,163 TS files in apps/desktop/src (162.6k LOC: components ~110k, lib ~40k) + rust 30.1k + crates (graph-paths w/ obsidian.rs compat, note-policy, index-schema) + extension + native-host + cli |
| Activity | no git history in download; release-please configured |
| Tests | 384 colocated test files; CI: ci, fix, pr-title, release, release-dmg, release-please, testflight |
| Entry points | `apps/desktop/src/app.tsx` (graph-chooser gate), `apps/desktop/src-tauri/src/main.rs` |

## Architecture
- **app.tsx** — boot gate: graph chooser ↔ GraphWorkspace; `installQuitFlush()` persists dirty buffers on quit/hide paths that skip unmount. [overview](scan-overview.html)
- **components/ (122 dirs)** — agents (routines runner, new-agent dialog), chat, browser, canvas, tasks, tags, settings (mcp-section), vault-compat/replace, terminal, backlinks, bases.
- **lib/ (185 files)** — controllers: backup, capture, chat-events, graph-layout + graph-frame-scheduler, github-*, csv, dates, browser-session, background-reconciler.
- **providers/** — React contexts: chat-provider (deliver/delete), capture, audio-memo, asset-describe.
- **src-tauri (43 modules, 30.1k)** — agent_cli.rs, browser.rs, calendar, capture, db, embed*.rs (embeddings incl. mobile), git, icloud, conflicts, contacts, process_memory/tree, liquid_glass.
- **crates/** — graph-paths (incl. obsidian.rs compatibility walking), index-schema, note-policy (frontmatter/heading).

## Quality evidence
- Builds / type-checks: not run (bun/cargo fetch needs network; `tauri` scripts + xcode env generation present).
- Tests: ✅ 384 colocated test files alongside their sources — a strict colocate-and-test habit across components and lib.
- Structure: ⚠️ components+lib hold ~150k LOC; provider/context pattern and per-concern stores keep it navigable but the surface is big.
- Error handling: sync-conflict-notice, backup-controller/reconciler, conflict module in rust — data-safety paths are explicit.
- Docs: ✅ README's "what Kore adds" list maps to code (tabs ↔ tab-pill.ts, rails, graph, browser.rs, agent_cli.rs, automations ↔ agent-routines-runner).
- Dependencies: React 19, Tauri 2; reflect-open inheritance acknowledged in README.
- Maintenance: not checked (no git history); release-please + dmg/testflight pipelines.

## Slop signals
- [ ] README promises features that have no code — *not fired*
- [ ] Large generated-looking files — *not fired*
- [ ] Dead code / placeholder impls — *not fired*
- [ ] Inconsistent architecture — *not fired*
- [x] No history in download — zip without `.git`
- [x] Missing LICENSE — MIT upstream, no license file in this fork (flagged, unknown)
- [ ] Doesn't build — *not fired* (not verified)
- [ ] Tests that assert nothing — *not fired*

## Verdict rubric
**good** — real colocated tests, thoughtful privacy design, clear fork delta; 2 slop signals (no history, missing license file).

## Worth stealing
- **Quit-flush persistence**: `installQuitFlush()` handles ⌘Q/secondary-window-close paths where unmount effects don't run (`apps/desktop/src/lib/quit-flush.ts`).
- **Private notes at sandbox level**: agent runs hard-block `private: true` notes in the CLI sandbox, not by prompt (`src-tauri/src/agent_cli.rs`).
- **Colocated test habit**: every component/controller ships with its `.test.tsx` — 384 of them.
- **Soul + memory agent files**: agent identity and memory as plain files beside a shared signed-facts profile.
- **Design tokens as a package**: design-system/ with theme tokens and an accent-color pipeline.

## Structural findings
1. **components/ + lib/ are ~150k LOC** — provider/context pattern holds it together; sub-domain packages (chat, graph, agents) would be the next cut. (warn)
2. **Agent privacy boundary is enforced in rust** — the CLI sandbox, not prompts, blocks private notes; trust boundary sits in agent_cli.rs. (info)
3. **Graph-paths crate keeps Obsidian compat** — `crates/graph-paths/src/obsidian.rs` walks/normalizes Obsidian-style vaults. (info)

## Scans
| Scope | JSON | Map |
|---|---|---|
| overview | [scan-overview.json](scan-overview.json) | [scan-overview.html](scan-overview.html) |
