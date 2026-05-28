# Orchestration: workflow-relocation

## Workflow ID
`0001-WORKFLOW-RELOCATION`

## Objective
Refactor hive-deck-pro so workflow definitions own the repo link. Remove `workflow_folder` from deck yaml and `WorkflowFolder` from `DeckFile`. Add `repo` to `WorkflowDef`. Update `workflowCmd` to resolve `{{WorkflowFolder}}` from the workflow's repo field, validating it is present in the deck. Add validation in `validate.go`.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0001-WORKFLOW-RELOCATION-0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [x] `Tasks/0001-WORKFLOW-RELOCATION-0001-TASK.md` — **Phase 1**: Add `Repo` to `WorkflowDef`, remove `WorkflowFolder` from `DeckFile`
- [x] `Tasks/0001-WORKFLOW-RELOCATION-0002-TASK.md` — **Phase 2**: Update `workflowCmd` to resolve repo from workflow definition
- [x] `Tasks/0001-WORKFLOW-RELOCATION-0003-TASK.md` — **Phase 3**: Add validation in `validate.go`
- [x] `Tasks/0001-WORKFLOW-RELOCATION-0004-TASK.md` — **Phase 4**: Update config files (`workflows.yaml`, deck yaml, examples)
- [x] `Tasks/0001-WORKFLOW-RELOCATION-0005-TASK.md` — **Final phase**: write Results + Retro/LESSONS, then hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/0001-WORKFLOW-RELOCATION-FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in Orchestrate/0001-WORKFLOW-RELOCATION-ORCH.md. Start by running hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/0001-WORKFLOW-RELOCATION-FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/0001-WORKFLOW-RELOCATION-RESULT.md and Retro/0001-WORKFLOW-RELOCATION-LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/0001-WORKFLOW-RELOCATION-RESULT.md`
- `Retro/0001-WORKFLOW-RELOCATION-LESSONS.md`
