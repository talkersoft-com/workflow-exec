# Lessons: agent-arcade/coarse-corgi

## What went well
- **Phase-per-PR with build + Node-logic verification** kept each step shippable and the deck
  always clean. Auto-merge + transition made the cadence smooth.
- **Freezing the IPC contract (0002) before either renderer** paid off — both the React Studio
  and the XState Arcade bound to a stable, documented seam, and the one verbatim-forwarding
  decision unblocked the event-driven dictation machine cleanly.
- The **plan flagged its own open questions**; the macro-authoring one turned out to be a
  non-existent reference surface — caught early instead of fabricated.

## What would have helped
1. **A headless Electron smoke harness.** The biggest verification gap every phase was "does the
   renderer actually run in Electron?" — subagents could only do build + jsdom/Node logic checks.
   A scripted `electron --run-once` that loads a renderer, asserts a few DOM nodes mounted, and
   exits would have let each phase self-verify rendering, not just bundling.
2. **A way to run live dictation/⌘W in CI/headless.** These need mic + the Spark backend + a real
   WezTerm pane; they can only be accepted on the operator's machine. Calling that out in the plan
   as an explicit "operator acceptance" task (separate from build tasks) would set expectations.
3. **computer-use permission pre-grant.** The final visual smoke was blocked on a macOS
   Accessibility/Screen-Recording grant; pre-granting before an exec run avoids the stall.

## Decisions worth remembering
- The dictation **machine observes** Go events; **main still routes** the pane send
  (additive, from 0002). Clean separation, no Go/contract changes — but two code paths set
  per-agent status (consistent, mild redundancy). A later pass could let the machine own the send.
- `recordingNavBehavior` shipped as an **optional additive** YAML field (default `send`), no
  schema break, no Studio UI yet — hand-editable. A Studio toggle is a small follow-up.
- `index.html`/CSS are byte-identical to the reference **except the one entry-script line** — the
  right reading of "reuse verbatim" (the bundle entry must differ).

## Fix files written
- None — no test failures required a FIX file; all phases passed their build + logic checks first try.
