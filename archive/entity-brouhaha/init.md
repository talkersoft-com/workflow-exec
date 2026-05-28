# Workflow: entity-brouhaha

## What this is

Update the `CloudManager.Entities` EF Core project and service/DTO layer to implement the playbook
management data model from `DATA-MODEL.md`. Remove all git-backed playbook code. Get the full API
building clean. Generate and apply the EF Core migration to local and remote databases. Ship.

## End state

- 9 new entity classes in `CloudManager.Entities`
- `Playbook.cs` and `PlaybookRun.cs` updated (git fields removed, new fields added)
- `CloudManagerDbContext` fully configured for all new/modified entities
- Public ID prefixes registered for all 9 new entity types
- DTO `Playbook` and `PlaybookRun` updated to match new entity shape
- `PlaybookService` git-backed methods removed (`RefreshAsync`, `DetectDriftForAssignmentAsync`)
- `PlaybookRunService.EnqueueAsync` updated to use `PlaybookId`
- `dotnet build CloudManager.API.csproj` exits with 0 errors
- EF migration `AddPlaybookManagement` generated and applied to local + remote DB
- `hv_ship` complete — PR open (and merged if auto_merge is on)
- Remote DB schema verified

## Inputs to read before any task

- `deck.md` — which hive-deck deck to use; **run `hv_status` + `hv_init` before Task 0001**
- `Orchestrate/0001-ENTITY-BROUHAHA-ORCH.md` — full task list and `/loop` directive
- `../instructions.md` — workflow conventions including hive-deck usage
- `../../planning/DATA-MODEL.md` — the authoritative schema this workflow implements

## The data model being implemented

Refer to `../../planning/DATA-MODEL.md` for the full ERD. Key changes:

### New entities (9)

| Class | Table | Schema | Notes |
|-------|-------|--------|-------|
| `PlaybookRevision` | `playbook_revisions` | `vm` | History of `Playbook.Content`; sequential `RevisionNumber` per playbook |
| `AnsibleRole` | `ansible_roles` | `vm` | Global shared role library; `Name` globally unique |
| `RoleFile` | `role_files` | `vm` | Files within a global role; `FilePath` unique per role |
| `RoleFileRevision` | `role_file_revisions` | `vm` | History per global role file |
| `PlaybookGlobalRoleRef` | `playbook_global_role_refs` | `vm` | Join: `Playbook` ↔ `AnsibleRole` |
| `PlaybookRole` | `playbook_roles` | `vm` | Role scoped to one playbook; `Name` unique per playbook |
| `PlaybookRoleFile` | `playbook_role_files` | `vm` | Files within a playbook-local role |
| `PlaybookRoleFileRevision` | `playbook_role_file_revisions` | `vm` | History per playbook-local role file |
| `PlaybookRunTarget` | `playbook_run_targets` | `vm` | Per-VM execution record within a `PlaybookRun` |

### Modified entities (2)

**`Playbook`** — remove git-backed fields, add DB-stored content:
- REMOVE: `GitUrl`, `GitRef`, `PlaybookPath`, `LastRefreshedAt`
- ADD: `Content` (string → `text NOT NULL`)

**`PlaybookRun`** — decouple from single assignment, add traceability:
- REMOVE: `AssignmentId`
- ADD: `PlaybookId`, `PlaybookRevisionId` (nullable), `VarsSnapshot` (jsonb), `Output` (text nullable)

## Pattern to mirror

Mirror the existing `Playbook`, `PlaybookRun`, and `VmPlaybookAssignment` config blocks in
`CloudManagerDbContext.cs`. Same FK style (`OnDelete(DeleteBehavior.Restrict)`), same jsonb/text
column type declarations, same index conventions.

## Naming conventions

- C# properties: PascalCase
- DB columns: `snake_case` via `.HasColumnName(...)`
- DB tables: `snake_case` via `.ToTable("name", "vm")`
- All `text` and `jsonb` columns set `.HasColumnType(...)` explicitly
- All entities inherit `Audit`

## Public ID prefixes for new entities

| Entity class | Prefix |
|---|---|
| `PlaybookRevision` | `pbrev` |
| `AnsibleRole` | `ansr` |
| `RoleFile` | `rf` |
| `RoleFileRevision` | `rfrev` |
| `PlaybookGlobalRoleRef` | `pgr` |
| `PlaybookRole` | `plr` |
| `PlaybookRoleFile` | `plrf` |
| `PlaybookRoleFileRevision` | `plrfr` |
| `PlaybookRunTarget` | `prt` |
