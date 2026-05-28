# Task: Update Playbook.cs and PlaybookRun.cs

## Task ID
`0002-ENTITY-BROUHAHA-TASK`

## Repo
`cloud-manager-api` (on branch `entity-brouhaha`)

## Objective
Remove git-backed fields from `Playbook` and decouple `PlaybookRun` from a single assignment.
These are breaking changes to the entity model — the DbContext config (Phase 3) and the migration
(Phase 4) will clean up the DB side.

## Files

### `src/Models/CloudManager.Entities/Models/Playbook.cs`

**REMOVE** these properties entirely:
- `public string GitUrl { get; set; }`
- `public string GitRef { get; set; }`
- `public string PlaybookPath { get; set; }`
- `public DateTime? LastRefreshedAt { get; set; }`

**ADD** this property (after `Description`):
```csharp
public string Content { get; set; } = string.Empty;
```

Keep `ArgumentSchema`, `OutputSchema` unchanged.

### `src/Models/CloudManager.Entities/Models/PlaybookRun.cs`

**REMOVE** this property entirely:
- `public Guid AssignmentId { get; set; }`

**ADD** these properties (after the existing `Id`):
```csharp
public Guid PlaybookId { get; set; }
public Guid? PlaybookRevisionId { get; set; }
```

**ADD** these properties (after `Stats`):
```csharp
public string VarsSnapshot { get; set; } = "{}";
public string? Output { get; set; }
```

Keep `Status`, `StartedAt`, `CompletedAt`, `Stats`, `VaultSecretsPrefix`,
`SecretsPublished`, `ErrorMessage` unchanged.

## Acceptance Criteria
- `Playbook.cs` has `Content` and does NOT have `GitUrl`, `GitRef`, `PlaybookPath`, or `LastRefreshedAt`.
- `PlaybookRun.cs` has `PlaybookId`, `PlaybookRevisionId`, `VarsSnapshot`, `Output` and does NOT have `AssignmentId`.
- `dotnet build src/Models/CloudManager.Entities/CloudManager.Entities.csproj` exits with 0 errors.

## Test
`Test/0002-ENTITY-BROUHAHA-TEST.md`

## On Failure
- Build errors referencing removed properties: those will surface in the DbContext (Phase 3) and
  services. That is expected — the DbContext references these properties for mapping. Fix them in
  Phase 3 rather than working around it here.
- If the build fails because `PlaybookService.cs` or other services reference the removed git
  fields, note the files but do NOT fix them in this phase. Service updates are a future workflow.
  The entity project (`CloudManager.Entities.csproj`) should build clean; the API project
  (`CloudManager.API.csproj`) may have errors until Phase 3 updates the DbContext.
