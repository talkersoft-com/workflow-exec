# Orchestration: agent-arcade/coarse-corgi

## Objective
Rebuild `agent-arcade-studio` into the `agent-arcade` repo: vanilla-JS Arcade (XState + mitt),
React Studio (Zustand), Go binaries + Electron main/preload/launcher + npm pipeline preserved
(one esbuild JS stage added; no DMG). Done = a publishable `@talkersoft-com/agent-arcade` whose
every menu, shortcut, and surface matches the reference 1:1, with durable per-agent drafts and
event-driven Go integration, verified by a parity pass + release smoke test.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../workflow-plans/agent-arcade/coarse-corgi/PLAN.md` (approved plan — source of truth)
- Reference: `/Users/talker/workspace/agent-arcade/agent-arcade-studio` (read-only)

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Scaffold + pipeline + Go + Electron shell (app launches, stubbed renderers) — build green; pack verified; TC-002 GUI launch deferred to 0007 smoke test
- [x] `Tasks/0002-TASK.md` — **Phase 2**: Freeze the preload / IPC contract (incl. Go NDJSON seam) — docs/IPC.md written; Arcade now forwards Go NDJSON verbatim (dictation:event + onDictationEvent); wezterm exit codes surfaced
- [x] `Tasks/0003-TASK.md` — **Phase 3**: Studio (React + Zustand), 1:1 against the same YAML — all reference screens rebuilt; CSS verbatim; no schema/main changes. NOTE: reference Studio has NO @-macro authoring UI (hand-edited YAML), so that test sub-item is N/A, not built. Live 1:1 deferred to 0007.
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Arcade foundation (machines + mitt + render + durable draft) — root machine + per-agent actors (durable draft, Node-verified), nav guards, mitt bus, IPC→XState seam, subscribe→DOM; index.html verbatim-except-entry; build green. CARRY TO 0007 PARITY: warn-on-exit/exit-confirm overlay, first-run tour, ? help overlay, rec-indicator not yet driven.
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: Arcade dictation + recording-nav (send/lock, async-commit)
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: Arcade terminal surface (peek, sync, ⌘W shell, macros, ^C)
- [ ] `Tasks/0007-TASK.md` — **Phase 7**: Parity pass + release smoke test; write Results + Retro/LESSONS, then hv_integrate → hv_release

## Execution steps
1. Read all inputs above before starting Task 0000.
2. For each unchecked task in order:
   a. Read the task file.
   b. Do the work (consult the reference repo for exact UI/behavior parity).
   c. Run the matching `Test/NNNN-TEST.md`.
   d. On failure: write `Retro/FIX-NNN.md`, apply the fix, re-run the test.
   e. On pass: check the box, ship the phase via `hv_integrate` (see `../deck.md`), move on.
3. When every box is checked, the workflow is complete.

## Autonomous execution
```
hv_orchestrate_run  deck: "agent-arcade"  branch: "coarse-corgi"
```
Or type:  `exec "coarse-corgi"`

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, …).
- Never silently retry — write the FIX file first, then apply the fix.
- If a failure cannot be recovered after two attempts: stop and surface to the operator.

## End-of-workflow outputs (write BEFORE the final hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
