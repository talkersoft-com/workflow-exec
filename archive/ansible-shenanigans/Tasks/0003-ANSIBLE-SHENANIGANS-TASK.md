# Task: Playbook data model

## Task ID
`0003-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Ship the schema for `vm.playbooks`, `vm.vm_playbook_assignments`, `vm.playbook_runs` exactly as specified in the plan, with prefix-registry + enum support, and an EF Core migration that applies cleanly.

## Context
- Column-by-column shapes are in `PLAYBOOK-INTEGRATION-PLAN.md` → "Data Model Additions". Don't re-quote here — read it.
- All three tables live in the `vm` schema (matches the plan; matches where `virtual_machines` and `snapshots` already live).
- All three inherit `Audit`. Don't re-implement the audit columns; the base entity handles them.

## Steps

1. **Enum**: add `PlaybookRunStatus { None, Queued, Running, Succeeded, Failed, Cancelled }` in `CloudManager.Entities/Enums/`. Register with `modelBuilder.HasPostgresEnum<PlaybookRunStatus>()` in `CloudManagerDbContext`.
   - Note: `vm_playbook_assignments.last_run_status` uses the subset `(None, Running, Succeeded, Failed)` per the plan. Reuse the same enum and document that `Queued` and `Cancelled` aren't expected to land in `assignments.last_run_status` (those are run-table-only states).

2. **Entities**
   - `Playbook` — Name (unique), Description, GitUrl, GitRef, PlaybookPath, ArgumentSchema (jsonb), OutputSchema (jsonb nullable), LastRefreshedAt (nullable timestamptz).
   - `VmPlaybookAssignment` — VirtualMachineId FK, PlaybookId FK, VarsOverride (jsonb), LastRunId FK (nullable), LastRunStatus (enum, default None), LastAppliedAt (nullable timestamptz). Unique constraint `(VirtualMachineId, PlaybookId)`.
   - `PlaybookRun` — AssignmentId FK, Status (enum), StartedAt, CompletedAt (nullable), Stats (jsonb), VaultSecretsPrefix (varchar 512), SecretsPublished (jsonb), ErrorMessage (text nullable).
   - FK delete behavior: `Restrict` everywhere (match existing convention from `vm.snapshots`).

3. **EntityPrefixRegistry** additions:
   ```csharp
   [typeof(Playbook)] = "pb",
   [typeof(VmPlaybookAssignment)] = "asgn",
   [typeof(PlaybookRun)] = "run",
   ```

4. **Migration**: `dotnet ef migrations add AddPlaybooks` (or use `scripts/database/add-migration.py`). Inspect generated SQL; confirm:
   - Three tables in `vm` schema
   - All audit columns present
   - Unique constraint on assignments
   - FK cascade behavior = Restrict
   - The new postgres enum is created

5. **Apply**: `dotnet ef database update` (or the project's wrapper). Confirm via psql.

6. **Skeleton service interfaces** (just interfaces — no impl bodies yet, Phase 0004 fills them in):
   - `IPlaybookService` with method signatures matching the API surface in the plan.
   - `IPlaybookAssignmentService`, `IPlaybookRunService` likewise.

## Acceptance Criteria
- `\dt vm.*` in psql shows the three new tables.
- `\d vm.playbooks` shows the columns and types from the plan.
- `\d vm.vm_playbook_assignments` shows the unique constraint.
- `\dT public.playbook_run_status` (or wherever the enum lives) shows the six values.
- `EntityPrefixRegistry` test (if it exists; otherwise `dotnet build`) passes.

## Test
`Test/0003-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
Migration name collisions, enum already-created errors from a prior partial run — typical EF Core foot-guns. Improvise specifically about which `dotnet ef` command was used, not a generic "migration failed."
