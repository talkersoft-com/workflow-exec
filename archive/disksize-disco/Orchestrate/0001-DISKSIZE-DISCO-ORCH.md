# Orchestration: disksize-disco

## Workflow ID
`0001-DISKSIZE-DISCO`

## Objective
User selects a disk size (GB) on the Create VM page → backend persists it → vorch-service resizes the qcow2 during provision + grows the partition → new VM boots with the requested disk. Mirror the existing `memory` field's plumbing exactly.

## Inputs

Read these before Task 0001. **Do not skip the branches.md step.**

- `../branches.md` — branch every repo listed before any code work
- `../init.md` — workflow entry point (this workflow's scope + naming choices)
- `../../instructions.md` — workflow conventions
- `../../craft-ingot/` — canonical file examples

## Task list

Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-DISKSIZE-DISCO-TASK.md` — **Phase 0**: Branch every repo in `branches.md` (no code yet)
- [x] `Tasks/0001-DISKSIZE-DISCO-TASK.md` — **Phase 1**: API entity + migration (`DiskSizeGb` int column on `vm.virtual_machines`)
- [x] `Tasks/0002-DISKSIZE-DISCO-TASK.md` — **Phase 2**: API DTO + AMQP message + Controller (plumb through, validate min/max)
- [x] `Tasks/0003-DISKSIZE-DISCO-TASK.md` — **Phase 3**: vorch-service handler — qemu-img resize + cloud-init growpart hook
- [x] `Tasks/0004-DISKSIZE-DISCO-TASK.md` — **Phase 4**: MCP `vm_create` adds `disk_size_gb` param
- [x] `Tasks/0005-DISKSIZE-DISCO-TASK.md` — **Phase 5**: Web form field on CreateVmPage + types + api client
- [x] `Tasks/0006-DISKSIZE-DISCO-TASK.md` — **Phase 6**: End-to-end smoke (create `disksize-test-1` with 10 GB; verify via SSH `lsblk` + `df -h /`)

## Orchestration steps

1. Read all inputs above before starting Task 0000
2. **Run Task 0000 first** — branch every repo. Skipping this is a process failure
3. For each remaining task in order:
   1. Read the task file
   2. Implement the work on the appropriate repo's `disksize-disco` branch
   3. Run the matching test file
   4. On failure, write `Improvise/000N-DISKSIZE-DISCO-IMPROVISE-NNN.md` and retry
   5. When the test passes, edit this orchestration in place to check the box
   6. Move to the next task
4. When every box is checked, write the workflow-level Result and Wishlist (see below) and stop

## Autonomous execution

```
/loop Continue executing tasks in this orchestration file. For each unchecked task in order: read the task file at the referenced path, do the work (Task 0000 = branch every repo per branches.md before any code), run the matching Test/<n> file, write an Improvise on failure and retry until pass, then edit this orchestration in place to check the box. When every box is checked, write Results/0001-DISKSIZE-DISCO-RESULT.md (workflow-level summary across all phases) and SelfImprove/0001-DISKSIZE-DISCO-WISHLIST-001.md (workflow-level reflection — MCPs, docs, access that would have compounded across phases) and stop.
```

## Improvisation policy

- One Improvise file per distinct failure mode encountered, numbered sequentially (`IMPROVISE-001`, `IMPROVISE-002`, ...)
- Reference the task and test case that failed
- Do not silently retry — write first, then retry
- A bug in an earlier phase discovered while running a later one: fix inline + write Improvise noting "discovered in Phase N, originated in Phase M"

## End-of-workflow outputs

- `Results/0001-DISKSIZE-DISCO-RESULT.md` — workflow-level result, phase-by-phase summary, smoke VM ID left up for inspection, the final state on each branch
- `SelfImprove/0001-DISKSIZE-DISCO-WISHLIST-001.md` — workflow-level reflection: what kept coming up across phases, what MCP / doc / access would have cut wall-clock the most

## References
- Tasks: `Tasks/0000-...` through `Tasks/0006-...`
- Tests: `Test/0001-...` through `Test/0006-...` (Phase 0 has no test; the branches.md verifies itself)
- Improvise: `Improvise/` (create on failure)
