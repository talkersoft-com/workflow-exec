# Result: cloud-manager/cosmic-strudel (neon-addax workflow)

## Outcome
SHIPPED — all 10 implementation tasks built clean; HTTP smoke tests deferred to manual verification (see `Retro/FIX-001.md`).

## Branch
`cosmic-strudel` (workflow folder name: `neon-addax` — workflow ID, not branch)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | TBD — filled from hv_ship output | merged |
| `execution-workflow` | TBD — filled from hv_ship output (Results + Lessons + FIX) | merged |

All other deck repos skipped (no changes).

## Phase summary

### Phase 0 — Initialize
hv_status: 14/14 clean on `cosmic-strudel`. Recorded in deck.md.

### Phase 1 — Migration `AddAnsibleConstraintsAndTargetTimestamps`
Discovered during inspection that the two unique indexes called for in `DATA-MODEL-DELTAS.md` are already declared in `CloudManagerDbContext.OnModelCreating`. The generated migration therefore only added the two new columns (`vm.playbook_run_targets.started_at`, `completed_at`), as expected. Migration applied to `clouddb` via `dotnet ef database update`. Verified live via `db_get_table_schema`.

### Phase 2 — AnsibleRole + RoleFile CRUD (API Phase 1)
DTOs: `AnsibleRole`, `RoleFile`, `RoleFileSummary`. Services: `IAnsibleRoleService` / `AnsibleRoleService`, `IRoleFileService` / `RoleFileService`. Path validator: `RoleFilePathValidator` (shared). Profile: `AnsibleRoleProfile`. Controllers: `AnsibleRoleController`, `RoleFileController`. DI in `ConfigureServices.cs`. `dotnet build` clean.

### Phase 3 — Revision write-side + read routes (API Phase 2)
DTOs: `RoleFileRevision`, `RoleFileRevisionSummary`. `RoleFileService.UpdateAsync` now appends a `RoleFileRevision` row in the same transaction with `revision_number = max + 1`. New `IRoleFileRevisionService` with restore endpoint that re-enters the auto-revision path (never deletes history). New `RoleFileRevisionController`. `RoleFileController.UpdateRoleFileRequest` extended with optional `change_summary`. `dotnet build` clean.

### Phase 4 — PlaybookRevision auto-create + read routes (API Phase 3)
DTOs: `PlaybookRevision`, `PlaybookRevisionSummary`. `PlaybookService.UpdateAsync` extended with `changeSummary` parameter; writes a `PlaybookRevision` row when content actually changes. `IPlaybookService` / `IPlaybookRunService` signatures updated. New `PlaybookRevisionController`. AutoMapper profile entries added. `dotnet build` clean.

### Phase 5 — PlaybookGlobalRoleRef + ArgumentSpecsTranslator hook (API Phase 4)
New `ArgumentSchemaRebuilder` helper merges JSON Schemas across all referenced roles for a playbook. Called from:
- `RoleFileService.UpdateAsync` when `file_path == "meta/argument_specs.yml"` (recomputes for every playbook referencing the role).
- `PlaybookGlobalRoleRefService.AttachAsync` / `DetachAsync` (recomputes for the affected playbook).
New DTO `PlaybookGlobalRoleRef`. New service + controller. Idempotency: attach rejects duplicates with 409. `dotnet build` clean.

### Phase 6 — Playbook-local roles (API Phase 5)
DTOs: `PlaybookRole`, `PlaybookRoleFile`, `PlaybookRoleFileSummary`, `PlaybookRoleFileRevision`, `PlaybookRoleFileRevisionSummary`. Three services in one file (`PlaybookRoleServices.cs`): `PlaybookRoleService`, `PlaybookRoleFileService`, `PlaybookRoleFileRevisionService`. Three controllers in one file (`PlaybookLocalRoleControllers.cs`). Argument-specs hook fires for the owning playbook only on local role file changes. Cascade delete on `PlaybookRole`. `dotnet build` clean.

### Phase 7 — PlaybookRunTarget reads (API Phase 6)
DTOs: `PlaybookRunTarget`, `PlaybookRunTargetSummary` (with `started_at` / `completed_at` from Phase 1 migration). Service joins VMs for name display. Controller exposes GET (list summaries), GET/{id} (full), and PATCH/{id} (worker writes). `dotnet build` clean.

### Phase 8 — Materialization endpoint (API Phase 7)
New `IPlaybookMaterializationService` + impl. Walks the playbook content + all referenced global roles + all playbook-local roles into a `files[]` array under `roles/{role.name}/{file.file_path}`. Local roles win on name collisions (matches ansible convention). Parses every `meta/argument_specs.yml` for `x-secret-path` annotations and emits a `secret_inputs[]` manifest of `{ extravar_name, vault_path }` for the worker. Revision query param uses historic playbook content while role files come from current head. New `GET /api/v1/playbook/{id}/materialized` endpoint on `PlaybookController`. `dotnet build` clean.

### Phase 9 — Multi-VM run-trigger endpoint (API Phase 8)
`IPlaybookRunService.TriggerMultiVmAsync` creates one `PlaybookRun` + N `PlaybookRunTarget` rows in one transaction. Per-VM logic: if a `VmPlaybookAssignment` exists for (vm, playbook), inherit its `vars_override` and set `assignment_id`; otherwise use the request's `varsOverride` (or `{}`) and leave `assignment_id = null`. After commit, publishes one message via existing `IPlaybookRunPublisher`. New endpoint `POST /api/v1/playbook/{id}/run`. Single-VM apply path on `PlaybookAssignmentController` preserved unchanged. Validation: empty `vmIds` → 400; unknown vmId → 400; empty playbook content → 400. `dotnet build` clean.

### Phase 10 — Run cancel + IPlaybookRunCancelPublisher (API Phase 9)
New `IPlaybookRunCancelPublisher` + `PlaybookRunCancelPublisher` (mirrors existing pattern; queue name exactly `playbook-run-cancel`). Registered in DI. `IPlaybookRunService.CancelAsync` (was stubbed) implemented: Queued → flip to Cancelled with `completed_at`; Running → leave alone (worker writes final state); already-terminal → 409. New endpoint `POST /api/v1/run/{runId}/cancel` on `RunController` that publishes the cancel message when the prior state was Running. `dotnet build` clean.

### Phase 11 — Ship
Two PRs produced by single `hv_ship`: `cloud-manager-api` (the implementation, ~30 new files + several edits) and `execution-workflow` (Results + Lessons + FIX-001). FIX-001 documents the HTTP/RabbitMQ smoke-test deferral.

## Files added (cloud-manager-api, ~30)

### DTOs (`src/Models/CloudManager.DTO/Models/`)
- `AnsibleRole.cs`, `RoleFile.cs` (+ `RoleFileSummary`), `RoleFileRevision.cs` (+ summary)
- `PlaybookRevision.cs` (+ summary)
- `PlaybookGlobalRoleRef.cs`
- `PlaybookRole.cs` (+ `PlaybookRoleFile`, `PlaybookRoleFileSummary`, `PlaybookRoleFileRevision`, `PlaybookRoleFileRevisionSummary`)
- `PlaybookRunTarget.cs` (+ summary)
- `MaterializedPlaybook.cs` (+ `MaterializedFile`, `SecretInputRef`)

### Service interfaces (`src/Services/CloudManager.Data.Services/Interface/`)
- `IAnsibleRoleService`, `IRoleFileService`, `IRoleFileRevisionService`
- `IPlaybookRevisionService`, `IPlaybookGlobalRoleRefService`
- `IPlaybookRoleService` (+ `IPlaybookRoleFileService`, `IPlaybookRoleFileRevisionService`)
- `IPlaybookRunTargetService`, `IPlaybookMaterializationService`

### Service impls (`src/Services/CloudManager.Data.Services/`)
- `AnsibleRoleService`, `RoleFileService`, `RoleFileRevisionService`
- `PlaybookRevisionService`, `PlaybookGlobalRoleRefService`
- `PlaybookRoleServices.cs` (3 services)
- `PlaybookRunTargetService`, `PlaybookMaterializationService`

### Helpers
- `Playbooks/RoleFilePathValidator.cs` (shared `..` / `/` / `\` / size guards)
- `Playbooks/ArgumentSchemaRebuilder.cs` (merges per-role JSON Schemas)

### AutoMapper profiles
- `AnsibleRoleProfile.cs`, `PlaybookRoleProfile.cs`; extended `PlaybookProfile.cs`

### Controllers (`src/CloudManager.API/Controllers/`)
- `AnsibleRoleController`, `RoleFileController`, `RoleFileRevisionController`
- `PlaybookRevisionController`, `PlaybookGlobalRoleRefController`
- `PlaybookLocalRoleControllers.cs` (3 controllers)
- `PlaybookRunTargetController`

### Publisher (`src/Services/CloudManager.AMQP.Publisher/`)
- `IPlaybookRunCancelPublisher.cs`, `PlaybookRunCancelPublisher.cs`

### EF
- Migration `20260528031302_AddAnsibleConstraintsAndTargetTimestamps` (just the 2 new timestamp columns; indexes already in DbContext + DB).

### Edits
- `PlaybookRunTarget.cs` entity gets `StartedAt`/`CompletedAt`
- `CloudManagerDbContext.cs` configures the new columns
- `IPlaybookService.UpdateAsync` signature extended with `changeSummary`
- `PlaybookService.UpdateAsync` writes `PlaybookRevision` on content change
- `RoleFileService.UpdateAsync` writes `RoleFileRevision` + fires argument-specs hook
- `IPlaybookRunService.CancelAsync` returns `Task<PlaybookRun>` + new `TriggerMultiVmAsync` method
- `PlaybookController` gets materialization + multi-VM run endpoints
- `RunController` gets cancel endpoint
- DI registrations in both `ConfigureServices.cs` files
