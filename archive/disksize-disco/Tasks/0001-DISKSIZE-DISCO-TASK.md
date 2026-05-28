# Task: API entity + migration

## Task ID
`0001-DISKSIZE-DISCO-TASK`

## Repo
`cloud-manager-api` (on branch `disksize-disco`)

## Objective
Add a `DiskSizeGb` integer column to the `VirtualMachine` entity, run the migration, and backfill existing rows.

## Pattern
Mirror `Memory`. Same int type, same column-mapping conventions in `CloudManagerDbContext`.

## Steps
1. Add `public int DiskSizeGb { get; set; }` to `src/Models/CloudManager.Entities/Models/VirtualMachine.cs`.
2. In `CloudManagerDbContext.cs`, add the property config in the existing `Entity<VirtualMachine>` block:
   ```csharp
   entity.Property(e => e.DiskSizeGb)
       .IsRequired()
       .HasColumnName("disk_size_gb")
       .HasDefaultValue(10);
   ```
3. Generate the migration: `python3 scripts/database/add-migration.py AddVmDiskSizeGb`
4. Edit the generated `Up()` to backfill existing rows AFTER `AddColumn`:
   ```csharp
   migrationBuilder.Sql("UPDATE vm.virtual_machines SET disk_size_gb = 10 WHERE disk_size_gb = 0;");
   ```
   (10 GB is a sane default for already-running test VMs; existing storage isn't actually larger but the column should reflect what we'd *want* if recreated.)
5. Apply: `python3 scripts/database/deploy-database.py`
6. Build the API: `dotnet build src/CloudManager.API/CloudManager.API.csproj` — confirm 0 errors.

## Acceptance Criteria
- `\d vm.virtual_machines` shows `disk_size_gb integer not null default 10`.
- All existing rows have a non-zero `disk_size_gb`.
- API build succeeds.

## Test
`Test/0001-DISKSIZE-DISCO-TEST.md`

## On Failure
- Migration name collision: name it `AddVmDiskSizeGbV2`.
- "default value 0 detected" warning: that's the int default-sentinel issue; fine to ignore, the SQL default + backfill cover it.
