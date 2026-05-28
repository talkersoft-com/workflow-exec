# Task: Remove git-backed service/DTO code — get full API build clean

## Task ID
`0004-ENTITY-BROUHAHA-TASK`

## Repo
`cloud-manager-api` (on branch `entity-brouhaha`)

## Objective
The entity and DbContext changes in Phases 1–3 removed fields that the DTO and service layers still
reference. This task removes all git-backed playbook code, updates the DTOs to match the new entity
shape, and gets `CloudManager.API.csproj` building with 0 errors. Nothing is stubbed — dead code
is deleted.

## Step 1 — Update DTOs

### `src/Models/CloudManager.DTO/Models/Playbook.cs`
- REMOVE: `GitUrl`, `GitRef`, `PlaybookPath`, `LastRefreshedAt`
- ADD: `public string Content { get; set; } = string.Empty;`

### `src/Models/CloudManager.DTO/Models/PlaybookRun.cs`
- REMOVE: `AssignmentId`
- ADD:
  ```csharp
  public string PlaybookId { get; set; } = string.Empty;
  public string? PlaybookRevisionId { get; set; }
  public string VarsSnapshot { get; set; } = "{}";
  public string? Output { get; set; }
  ```

## Step 2 — Update IPlaybookService + PlaybookService

### `src/Services/CloudManager.Data.Services/Interface/IPlaybookService.cs`
- REMOVE method signatures:
  - `Task<Playbook> RefreshAsync(string id);`
  - `Task<DriftResult> DetectDriftForAssignmentAsync(string playbookPublicId, string varsOverrideJson);`

### `src/Services/CloudManager.Data.Services/Playbooks/PlaybookService.cs`
- REMOVE the `RefreshAsync` implementation entirely
- REMOVE the `DetectDriftForAssignmentAsync` implementation entirely
- In `CreateAsync`: remove any call to `RefreshAsync` — just save the playbook entity and return the DTO
- In `UpdateAsync`: remove any call to `RefreshAsync` — just update and return

If `ArgumentSpecsTranslator` or `SchemaDiff` are only referenced from the removed methods, remove
those `using` statements too. Do NOT delete the files themselves — they will be re-wired in a
future workflow.

## Step 3 — Update IPlaybookRunService + PlaybookRunService

### `src/Services/CloudManager.Data.Services/Playbooks/PlaybookRunService.cs` — `EnqueueAsync`

The old implementation set `AssignmentId`. Replace with `PlaybookId`. The method receives an
`assignmentId` parameter — look up the assignment to get its `PlaybookId`, then set that on the
new run. Pattern:

```csharp
var assignment = await _context.VmPlaybookAssignments
    .FirstOrDefaultAsync(a => a.PublicId == assignmentId);
if (assignment is null) throw new KeyNotFoundException($"Assignment {assignmentId} not found");

var run = new PlaybookRun
{
    Id = Guid.NewGuid(),
    PlaybookId = assignment.PlaybookId,
    Status = PlaybookRunStatus.Queued,
    StartedAt = DateTime.UtcNow,
    Stats = "{}",
    VarsSnapshot = assignment.VarsOverride,
    VaultSecretsPrefix = string.Empty,
    SecretsPublished = "{}",
};
```

## Step 4 — Fix AutoMapper + any remaining compile errors

Run `dotnet build src/CloudManager.API/CloudManager.API.csproj` and address each remaining compile
error one at a time. Common sources:
- AutoMapper profiles mapping removed DTO fields: remove those explicit `.ForMember(...)` lines
- Controllers reading `AssignmentId` or git fields from request: remove or comment those lines
- AMQP publisher referencing `AssignmentId` on a run: remove or update

Do not add stubs or workarounds — delete the dead code path.

## Step 5 — Verify build
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/CloudManager.API/CloudManager.API.csproj 2>&1 | grep -E "Error\(s\)|Build succ" | tail -5
```
**Pass**: `Build succeeded` with `0 Error(s)`.

## Acceptance Criteria
- DTO `Playbook` has `Content`, no git fields
- DTO `PlaybookRun` has `PlaybookId`, no `AssignmentId`
- `IPlaybookService` no longer declares `RefreshAsync` or `DetectDriftForAssignmentAsync`
- `PlaybookRunService.EnqueueAsync` uses `PlaybookId`, not `AssignmentId`
- `dotnet build src/CloudManager.API/CloudManager.API.csproj` exits with `0 Error(s)`

## Test
`Test/0004-ENTITY-BROUHAHA-TEST.md`

## On Failure
- AutoMapper profile referencing removed field: search the entire `src/` tree for the field name and remove each reference
- Controller or publisher still referencing `AssignmentId`: remove the usage — no stub, no workaround
- Build errors in a file not anticipated here: write an Improvise, remove the dead reference, retry
