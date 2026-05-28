# Orchestration: cloud-manager/purple-vulture

## Objective
An operator can reach the playbook YAML editor from `/ansible/playbooks` (list) or `/ansible/playbooks/:pid` (detail), edit and save the YAML (creating a `playbook_revisions` row), then Close back to the page they came from — with no URL-typing and no dead-end pages. Cold-load of `/edit` falls back to the detail page on Close.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`
- `../../../workflow-plans/cloud-manager/gingery-kingfisher/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record execution branch
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Editor close button + referrer + `onDirtyChange` from `RoleFileEditor` + `beforeunload` + flex layout
- [x] `Tasks/0002-TASK.md` — **Phase 2**: Edit button on `PlaybookDetailPage` header + pencil icon on each `PlaybooksPage` row, both passing `location.state.from`
- [x] `Tasks/0003-TASK.md` — **Phase 3**: Typecheck + build + deploy via `install-web-app.py`
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Playwright verification of all three close paths (from list, from detail, cold-load)
- [x] `Tasks/0005-TASK.md` — **Phase 5**: Write `Results/RESULT.md` + `Retro/LESSONS.md`; then `hv_ship`

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
```
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/purple-vulture/Orchestrate/ORCH.md. Start by running
hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read
the task file, do the work, run the matching Test file. On failure write
Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked
write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship —
then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator
- **Do not expand scope.** No edits to `RoleFileEditor` internals beyond `onDirtyChange`. No edits to Run / History / role-detail pages. No backend changes.

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
