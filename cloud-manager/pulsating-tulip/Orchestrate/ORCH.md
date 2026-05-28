# Orchestration: cloud-manager/pulsating-tulip

## Objective
Restructure `ClaudeSettings` so `permissions` maps 1:1 to Claude's JSON schema. Ship to
warm-leviathan, then reprovision the deck so all repos get clean settings files.

## Inputs
- `../init.md`
- `../deck.md`

## Task list
- [ ] `Tasks/0000-TASK.md` — **Phase 0**: verify workspace
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: restructure Go structs + rewrite claude.go + update config files
- [ ] `Tasks/0002-TASK.md` — **Final**: build, install, commit, write Results + Retro/LESSONS, hv_ship, hv_init

## Execution steps
1. For each unchecked task: read it, do the work, run the matching Test, check the box
2. On failure: write `Retro/FIX-NNN.md`, fix, re-run
3. When all boxes checked, workflow is complete

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially
- Never silently retry — write the FIX file first

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
