# Orchestration: entity-brouhaha

## Workflow ID
`0001-ENTITY-BROUHAHA`

## Objective

Update `CloudManager.Entities`, `CloudManagerDbContext`, and the service/DTO layer to implement
the playbook management data model from `DATA-MODEL.md`. Remove all git-backed playbook code.
Get the full API project building clean. Generate and apply the EF Core migration. Ship.
Verify the database deployed on remote.

## Inputs

Read these before Task 0000. **Do not skip the hive-deck initialization step.**

- `../deck.md` — deck name and which repos have code changes; `hv_status` + `hv_init` before any code
- `../init.md` — full scope, entity table, naming conventions, public ID prefixes
- `../../instructions.md` — workflow conventions including hive-deck usage
- `../../../planning/DATA-MODEL.md` — authoritative ERD this workflow implements

## Task list

Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-ENTITY-BROUHAHA-TASK.md` — **Phase 0**: `hv_status` + `hv_init` — provision workspace and create feature branch
- [x] `Tasks/0001-ENTITY-BROUHAHA-TASK.md` — **Phase 1**: Add 9 new entity classes
- [x] `Tasks/0002-ENTITY-BROUHAHA-TASK.md` — **Phase 2**: Update `Playbook.cs` and `PlaybookRun.cs`
- [x] `Tasks/0003-ENTITY-BROUHAHA-TASK.md` — **Phase 3**: Update `CloudManagerDbContext` + public ID prefixes
- [x] `Tasks/0004-ENTITY-BROUHAHA-TASK.md` — **Phase 4**: Remove git-backed service/DTO code — get full API build clean
- [x] `Tasks/0005-ENTITY-BROUHAHA-TASK.md` — **Phase 5**: Generate EF migration + apply to local and remote DB
- [x] `Tasks/0006-ENTITY-BROUHAHA-TASK.md` — **Phase 6**: `hv_ship` + verify DB deployed on remote

## Orchestration steps

1. Read all inputs above before starting Task 0000
2. **Run Task 0000 first** — `hv_status` then `hv_init` (or `hv_next`). Skipping this is a process failure
3. For each remaining task in order:
   1. Read the task file
   2. Implement the work on `cloud-manager-api` (workspace is on the feature branch)
   3. Run the matching test file
   4. On failure, write `Improvise/0001-ENTITY-BROUHAHA-IMPROVISE-NNN.md` and retry
   5. When the test passes, edit this orchestration in place to check the box
   6. Move to the next task
4. When every box is checked, write Result and Wishlist — workflow complete

## Autonomous execution

```
/loop Continue executing tasks in this orchestration file. Start by running hv_status then hv_init (or hv_next) on the cloud-manager deck per deck.md. For each unchecked task in order: read the task file, do the work, run the matching Test file, write an Improvise on failure and retry until pass, then check the box. When every box is checked, write Results/0001-ENTITY-BROUHAHA-RESULT.md and SelfImprove/0001-ENTITY-BROUHAHA-WISHLIST-001.md and stop.
```

## Improvisation policy

- One Improvise file per distinct failure mode, numbered sequentially (`IMPROVISE-001`, `IMPROVISE-002`, ...)
- Reference the task and test case that failed
- Do not silently retry — write first, then retry
- Migration name conflict: use `AddPlaybookManagementV2`
- Build errors in unexpected files: remove/stub only what is needed to compile; note each in an Improvise

## End-of-workflow outputs

- `Results/0001-ENTITY-BROUHAHA-RESULT.md` — outcome, phase summary, migration name, auto-generated branch name, PR URLs, DB verification result
- `SelfImprove/0001-ENTITY-BROUHAHA-WISHLIST-001.md` — what would have helped across phases

## References

- Tasks: `Tasks/0000-...` through `Tasks/0006-...`
- Tests: `Test/0000-...` through `Test/0006-...`
- Improvise: `Improvise/` (create on failure)
- Data model: `../../../planning/DATA-MODEL.md`
- Deck: `../deck.md`
