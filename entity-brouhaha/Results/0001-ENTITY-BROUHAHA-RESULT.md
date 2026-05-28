# Result: entity-brouhaha

## Workflow ID
`0001-ENTITY-BROUHAHA`

## Outcome
**Complete.** All 7 phases executed successfully. The playbook management data model is
implemented, the API builds clean, and the EF Core migration is deployed to both databases.

## Branch
`leafy-sable`

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| planning | https://github.com/talkersoft-com/planning/pull/1 | auto-merged |
| execution-workflow | https://github.com/talkersoft-com/execution-workflow/pull/5 | auto-merged |
| cloud-manager-api | https://github.com/talkersoft-com/cloud-manager-api/pull/9 | auto-merged |

## Migration
**Name**: `AddPlaybookManagement`
**File**: `20260526220320_AddPlaybookManagement.cs`

## Phase Summary

### Phase 0 — hv_status + hv_init
Workspace already on `leafy-sable` feature branch. No init needed.

### Phase 1 — 9 new entity classes
Created all 9 entity files in `CloudManager.Entities/Models/`:
- `PlaybookRevision`, `AnsibleRole`, `RoleFile`, `RoleFileRevision`
- `PlaybookGlobalRoleRef`, `PlaybookRole`, `PlaybookRoleFile`, `PlaybookRoleFileRevision`
- `PlaybookRunTarget`

### Phase 2 — Update Playbook.cs and PlaybookRun.cs
- `Playbook`: removed `GitUrl`, `GitRef`, `PlaybookPath`, `LastRefreshedAt`; added `Content`
- `PlaybookRun`: removed `AssignmentId`; added `PlaybookId`, `PlaybookRevisionId`, `VarsSnapshot`, `Output`

### Phase 3 — Update CloudManagerDbContext + public ID prefixes
- Added 9 new `DbSet<T>` declarations
- Added 9 new entity configuration blocks (all in `vm` schema)
- Fixed `Playbook` and `PlaybookRun` config blocks to match Phase 2 changes
- Added 9 new prefix entries to `EntityPrefixRegistry`
- Improvise-001: TC-004 false positive — `PlaybookRunTarget.AssignmentId` (valid new field)
  triggered the grep, not the removed `PlaybookRun.AssignmentId`

### Phase 4 — Remove git-backed service/DTO code
- Updated `DTO/Playbook.cs` and `DTO/PlaybookRun.cs`
- Removed `RefreshAsync` and `DetectDriftForAssignmentAsync` from `IPlaybookService`
- Rewrote `PlaybookService` (removed all git-backed methods and helpers)
- Rewrote `PlaybookRunService` (EnqueueAsync now uses `PlaybookId` from assignment)
- Fixed `PlaybookProfile` AutoMapper, `PlaybookAssignmentService`, `PlaybookController`,
  `PlaybookAssignmentController`
- `dotnet build CloudManager.API.csproj` → 0 errors

### Phase 5 — Generate EF migration + apply to local and remote DB
- Generated `AddPlaybookManagement` migration via `dotnet ef migrations add`
- Improvise-002: 1 existing row in `vm.playbook_runs` required backfill SQL;
  EF renamed `assignment_id → playbook_id` (not drop+add), so backfill uses
  `JOIN vm_playbook_assignments` to get the real `playbook_id`
- Improvise-002: `defaultValue: ""` for JSONB column was invalid; changed to `"{}"`
- Migration applied to remote DB (port 5432) via design-time factory in `DbContextFactory.cs`
- Migration applied to local DB (port 5433) via `--connection` override flag
- Both DBs verified: 9 new tables, correct column schemas

### Phase 6 — hv_ship + verify DB deployed
- `hv_ship` committed and shipped 3 repos (planning, execution-workflow, cloud-manager-api)
- All PRs auto-merged
- Remote DB final verification: 9 new tables present, `content` on playbooks, `playbook_id`
  on playbook_runs, no git columns or `assignment_id`

## DB Verification (Remote — port 5432 via SSH tunnel)

| Check | Result |
|-------|--------|
| 9 new vm-schema tables | ✅ All present |
| `vm.playbooks` has `content` | ✅ |
| `vm.playbooks` has no git columns | ✅ |
| `vm.playbook_runs` has `playbook_id` | ✅ |
| `vm.playbook_runs` has no `assignment_id` | ✅ |

## DB Verification (Local — port 5433)

| Check | Result |
|-------|--------|
| 9 new vm-schema tables | ✅ All present |
| `vm.playbooks` has `content` | ✅ |
| `vm.playbook_runs` has `playbook_id` | ✅ |
