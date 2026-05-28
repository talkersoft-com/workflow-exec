# Task: Add 9 new entity classes

## Task ID
`0001-ENTITY-BROUHAHA-TASK`

## Repo
`cloud-manager-api` (on branch `entity-brouhaha`)

## Objective
Create C# entity class files for all 9 new entities in `src/Models/CloudManager.Entities/Models/`.
All classes inherit `Audit`. Properties must use the exact names below — DbContext configuration
in Phase 3 depends on them.

## Pattern
Mirror the existing `Playbook.cs` and `PlaybookRun.cs` in the same directory — same using
directives, same `Audit` base class, same namespace (`CloudManager.Entities.Models`).

## Files to create

### 1. `PlaybookRevision.cs`
```csharp
namespace CloudManager.Entities.Models;

public class PlaybookRevision : Audit
{
    public Guid Id { get; set; }
    public Guid PlaybookId { get; set; }
    public int RevisionNumber { get; set; }
    public string Content { get; set; } = string.Empty;
    public string? ChangeSummary { get; set; }
}
```

### 2. `AnsibleRole.cs`
```csharp
namespace CloudManager.Entities.Models;

public class AnsibleRole : Audit
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
}
```

### 3. `RoleFile.cs`
```csharp
namespace CloudManager.Entities.Models;

public class RoleFile : Audit
{
    public Guid Id { get; set; }
    public Guid RoleId { get; set; }
    public string FilePath { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
}
```

### 4. `RoleFileRevision.cs`
```csharp
namespace CloudManager.Entities.Models;

public class RoleFileRevision : Audit
{
    public Guid Id { get; set; }
    public Guid RoleFileId { get; set; }
    public int RevisionNumber { get; set; }
    public string Content { get; set; } = string.Empty;
    public string? ChangeSummary { get; set; }
}
```

### 5. `PlaybookGlobalRoleRef.cs`
```csharp
namespace CloudManager.Entities.Models;

public class PlaybookGlobalRoleRef : Audit
{
    public Guid Id { get; set; }
    public Guid PlaybookId { get; set; }
    public Guid AnsibleRoleId { get; set; }
}
```

### 6. `PlaybookRole.cs`
```csharp
namespace CloudManager.Entities.Models;

public class PlaybookRole : Audit
{
    public Guid Id { get; set; }
    public Guid PlaybookId { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
}
```

### 7. `PlaybookRoleFile.cs`
```csharp
namespace CloudManager.Entities.Models;

public class PlaybookRoleFile : Audit
{
    public Guid Id { get; set; }
    public Guid PlaybookRoleId { get; set; }
    public string FilePath { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
}
```

### 8. `PlaybookRoleFileRevision.cs`
```csharp
namespace CloudManager.Entities.Models;

public class PlaybookRoleFileRevision : Audit
{
    public Guid Id { get; set; }
    public Guid PlaybookRoleFileId { get; set; }
    public int RevisionNumber { get; set; }
    public string Content { get; set; } = string.Empty;
    public string? ChangeSummary { get; set; }
}
```

### 9. `PlaybookRunTarget.cs`
```csharp
using CloudManager.Entities.Enums;

namespace CloudManager.Entities.Models;

public class PlaybookRunTarget : Audit
{
    public Guid Id { get; set; }
    public Guid RunId { get; set; }
    public Guid VmId { get; set; }
    public Guid? AssignmentId { get; set; }
    public string VarsOverrideSnapshot { get; set; } = "{}";
    public PlaybookRunStatus Status { get; set; } = PlaybookRunStatus.Queued;
    public string Stats { get; set; } = "{}";
    public string? Output { get; set; }
}
```

## Acceptance Criteria
- All 9 files exist under `src/Models/CloudManager.Entities/Models/`
- `dotnet build src/Models/CloudManager.Entities/CloudManager.Entities.csproj` exits with 0 errors

## Test
`Test/0001-ENTITY-BROUHAHA-TEST.md`

## On Failure
- Namespace not found for `PlaybookRunStatus`: verify `using CloudManager.Entities.Enums;` is present in `PlaybookRunTarget.cs`.
- `Audit` base not found: check that the file has no `using` conflicts and matches the namespace of the existing entity files in the same directory.
