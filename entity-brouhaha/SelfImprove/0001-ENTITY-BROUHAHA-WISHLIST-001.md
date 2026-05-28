# Wishlist: entity-brouhaha

## Workflow ID
`0001-ENTITY-BROUHAHA`

## What would have helped

### 1. Document the design-time factory in deck.md
`DbContextFactory.cs` reads from `appsettings.json` (port 5432 / cloudmanager user) rather than
env vars, so `dotnet ef database update` always targets the remote DB by default. This is
non-obvious and caused confusion when trying to apply the migration to the local DB. The deck.md
(or task 0005) should explicitly state this and document the `--connection` override pattern.

### 2. Include `dotnet ef migrations add` env-var requirements in the task
The migration generation step requires env vars to satisfy the startup project's `Program.cs`
validation (DBHOST, DBUSER, etc.). The task referenced the Python script but the script runs from
the wrong working directory. Documenting the direct `dotnet ef` command with dummy env vars would
save a debug cycle.

### 3. Test TC-004 grep should exclude entity model files
Phase 4's TC-004 grep for `AssignmentId` matched `PlaybookRunTarget.AssignmentId` (a valid new
field). A better grep would filter to only the DTO and service layers:
```bash
grep -r "AssignmentId" src/ --include="*.cs" \
  --exclude-dir=Migrations --exclude-dir=Models/CloudManager.Entities/Models
```

### 4. JSONB `defaultValue` convention
EF Core's generated `defaultValue: ""` for a JSONB column is invalid. The migration post-processor
(or task instructions) should note that EF uses empty string as the default for `string` columns
and that JSONB columns require `"{}"` as the default value. This bit us on the first migration
attempt.

### 5. Note the RenameColumn behavior for EF type-inference
When a `Guid NOT NULL` property is renamed between migrations, EF infers a `RenameColumn` rather
than `DropColumn + AddColumn`. This preserves old data (assignment GUIDs) in the renamed column,
requiring a backfill JOIN. Task 0005 should warn about this behavior and provide the correct
backfill SQL pattern (joining on the old FK table rather than checking for empty GUIDs).
