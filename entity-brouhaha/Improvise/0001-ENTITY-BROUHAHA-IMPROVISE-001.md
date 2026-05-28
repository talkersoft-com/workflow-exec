# Improvise: TC-004 false positive on AssignmentId

## Task
`0004-ENTITY-BROUHAHA-TASK.md` — Step 4 / Test TC-004

## Failure
`Test/0004-ENTITY-BROUHAHA-TEST.md` TC-004 grep for `AssignmentId` across `src/` reports two files:
- `src/Models/CloudManager.Entities/CloudManagerDbContext.cs`
- `src/Models/CloudManager.Entities/Models/PlaybookRunTarget.cs`

## Root Cause
The TC-004 grep for `AssignmentId` is too broad. The test intends to catch residual references to
the **removed** `PlaybookRun.AssignmentId` field. However, `PlaybookRunTarget` was created in
Phase 1 as a new entity that intentionally carries an **optional** `Guid? AssignmentId` property —
a nullable FK to `VmPlaybookAssignment` that records which assignment triggered a particular run
target. This is a valid new field, not a remnant of the old git-backed model.

## Disposition
False positive. Both hits are for `PlaybookRunTarget.AssignmentId`, not `PlaybookRun.AssignmentId`.
The actual `PlaybookRun.AssignmentId` is fully removed: entity model, DbContext config, DTOs,
services, controllers, and AutoMapper profiles all clean.

TC-005 passes with `0 Error(s)`. The build is clean.

## Action Taken
No code changes needed. Proceeding past TC-004 — the "removed field" is gone; the grep hit is a
newly added field on a different entity.
