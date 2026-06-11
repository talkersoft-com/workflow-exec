# Execution: cloud-manager/copper-kiosk

## Objective

The Marketplace is usable by a person in cloud-manager-web: browse published blueprints at
`/marketplace` and create a VM from one in three clicks (host, name, go), landing on the VM detail
page to watch it provision and its playbook chain run live; compose and publish blueprints at
`/blueprints` (image + ordered playbooks + vars, Draft/Published/Archived lifecycle); VM detail
shows blueprint provenance and per-step chain progress. All of it invisible with the `marketplace`
feature flag off, and the existing wizard + `/ansible/*` pages byte-identical. One findings doc
answers whether vorch/porch status writebacks suffice for this UI (expected: yes — no code there).

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/copper-kiosk/PLAN.md`
- `../../lucky-engineblock/Results/RESULT.md` (what the live backend actually does)

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record the execution branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1**: plumbing — api client, Redux slices, flag wiring, gated routes + nav
- [x] `Tasks/0002-TASK.md` — **Phase 2**: marketplace storefront — cards + instantiate flow → VM detail
- [x] `Tasks/0003-TASK.md` — **Phase 3**: blueprint builder — compose, reorder, vars, publish/archive
- [x] `Tasks/0004-TASK.md` — **Phase 4**: VM provenance + live run-chain panel (SignalR)
- [x] `Tasks/0005-TASK.md` — **Phase 5**: vorch/porch status-surface assessment → findings doc only
- [x] `Tasks/0006-TASK.md` — **Final phase**: deploy web, full e2e + regression, Results + Retro/LESSONS, then hv_ship

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
hv_orchestrate_run  deck: "cloud-manager"  branch: "copper-kiosk"
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
