# Result: agent-arcade/coarse-corgi — Agent Arcade rebuild

## Outcome
**BUILT, PACKAGED, INSTALLS, AND LAUNCHES** across all 8 phases. **Pending live
acceptance** — full visual 1:1 parity + live dictation/⌘W round-trips against the
real backend were not exercised here (computer-use permission + the operator's live
environment required).

## Deck / orchestration
- Deck: `agent-arcade` · Orchestration: `coarse-corgi`
- Each phase shipped on its own auto-merged PR and transitioned the deck.

## Pull Requests
| Phase | Repo | PR | Status |
|------|------|----|--------|
| 0001 Scaffold + pipeline + Go + Electron shell | agent-arcade | #2 | merged |
| 0002 IPC contract freeze + event-driven Go seam | agent-arcade | #3 | merged |
| 0003 Studio (React + Zustand) | agent-arcade | #4 | merged |
| 0004 Arcade foundation (machines + durable draft) | agent-arcade | #5 | merged |
| 0005 Dictation + recording-nav | agent-arcade | #6 | merged |
| 0006 Terminal surface (peek/sync/⌘W/macros/^C) | agent-arcade | #7 | merged |
| 0007 Parity + smoke (this) | agent-arcade | (this PR) | — |

## Phase summary
- **0000 Init** — deck transitioned to a fresh execution branch; all repos clean.
- **0001** — `@talkersoft-com/agent-arcade` (bin `agent-arcade`) scaffolded: Go bridges +
  Electron main/preload/launcher/arcade-main copied; DMG/electron-builder dropped; esbuild
  `build:js` stage added (studio/dist + arcade/dist); node-pty postinstall kept. Full build
  green; pack verified.
- **0002** — `docs/IPC.md` frozen. Arcade main now forwards **every** dictation-go NDJSON
  message verbatim on `dictation:event` (+`onDictationEvent`); wezterm-bridge exit codes
  surfaced for optimistic-with-reconciliation.
- **0003** — Studio rewritten as a React 18 + Zustand SPA, 1:1 with every reference screen,
  CSS verbatim, same YAML, no schema/main changes. (Reference Studio has no macro-authoring
  UI, so none was invented.)
- **0004** — Arcade foundation: root machine + per-agent child actors (**durable draft** —
  the headline fix, Node-verified), nav guards, mitt bus, single IPC→XState seam,
  subscribe→DOM. `index.html`/CSS verbatim except the entry script line.
- **0005** — Dictation machine (per-`job_id` actor) resolves on Go's events;
  `recordingNavBehavior` send/lock + Esc-discard with guards; async-commit keeps the job
  bound to the **originating** agent across navigation (Node-verified).
- **0006** — Terminal surface as one `terminalMachine` (closed→peek→sync→shell) + controller:
  live peek, ^C, sync mode (keyEventToBytes 17/17), ⌘W node-pty+xterm shell, @-macro picker
  (select/text/flag; compose 8/8), optimistic-with-reconciliation on non-zero wez exit.
- **0007** — Release artifact verified: full `npm run build` green; `npm pack` (105 MB) →
  install into a clean temp prefix → `spawn-helper` executable (⌘W path) → bin present →
  all renderers + Go bridges + WezTerm bundled. **Live launch:** the installed app starts in
  real Electron (Studio mode) and stays alive with **no crash / no JS errors** (verified
  after clearing a single-instance-lock false start).

## What is verified vs pending
**Verified:** full pipeline build; pack/install; postinstall spawn-helper; the app launches
and runs without crashing in real Electron; per-phase build + Node-level logic (durable draft,
async-commit, macro compose, keyEventToBytes, machine transitions). Paradigm boundary holds
(React only in `studio/`; Arcade is vanilla + XState + mitt).

**Pending live acceptance (needs the operator + computer-use grant + the real backend):**
visual 1:1 parity per surface; a real dictation round-trip (mic + Spark backend); a real ⌘W
node-pty shell + xterm render; the carry-forward parity items from §PARITY.

## Release status
Code promoted hive→main via `hv_release`. The **npm publish (cutting `v0.1.0`) is intentionally
deferred** until live acceptance passes — the new package has never been published, and the
first release should follow the operator's live sign-off.
