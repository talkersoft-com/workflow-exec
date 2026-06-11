# Execution: hive-deck-pro/drifting-toadstool

## Objective

hv supports two config truths with full back-compat: machine config.yaml (identity, never
inherited) optionally declares `deck_config:` pointing at a shared layer; decks, workflow
extensions, fragments, modules, claude-profiles, and gitignore-rulesets resolve machine-first then
layer per file; `hv decks`, the workflow registry, and fragment lookup union both layers (machine
wins). `{{WorkspaceRoot}}` works everywhere paths are declared (mcps.yaml included), with
`{{DecksRoot}}` as a living alias, `~` expansion, and relative-to-declaring-file semantics.
A config WITHOUT `deck_config` produces byte-identical behavior to today — proven by golden
comparison.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/hive-deck-pro/drifting-toadstool/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record the execution branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1**: token engine — expandPath + config path keys + mcps.yaml tokens
- [x] `Tasks/0002-TASK.md` — **Phase 2**: shared layer — Setup.DeckConfig, layerPaths, per-file fallback, union scans
- [x] `Tasks/0003-TASK.md` — **Phase 3**: machine identity via HV_PROFILE + docs
- [x] `Tasks/0004-TASK.md` — **Final phase**: golden back-compat + two-layer matrix, rebuild artifacts, Results + Retro/LESSONS, then hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "hive-deck-pro"  branch: "drifting-toadstool"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
