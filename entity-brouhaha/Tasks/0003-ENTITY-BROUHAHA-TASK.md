# Task: Update CloudManagerDbContext + public ID prefixes

## Task ID
`0003-ENTITY-BROUHAHA-TASK`

## Repo
`cloud-manager-api` (on branch `entity-brouhaha`)

## Objective
1. Add `DbSet<T>` declarations for all 9 new entities in `CloudManagerDbContext`
2. Configure all 9 new entities in `OnModelCreating` (table, schema, columns, FKs, indexes)
3. Update the existing `Playbook` and `PlaybookRun` configurations to match Phase 2 changes
4. Register public ID prefixes for all 9 new entity types

## Pattern
Find the existing `Playbook`, `PlaybookRun`, and `VmPlaybookAssignment` config blocks in
`CloudManagerDbContext.cs` — mirror their style exactly. All FKs use
`OnDelete(DeleteBehavior.Restrict)`. All jsonb columns use `.HasColumnType("jsonb")`. All text
columns use `.HasColumnType("text")`.

## Step 1 — DbSet declarations

Add to the DbSet block (grouped with the other playbook entities):
```csharp
public DbSet<PlaybookRevision> PlaybookRevisions { get; set; }
public DbSet<AnsibleRole> AnsibleRoles { get; set; }
public DbSet<RoleFile> RoleFiles { get; set; }
public DbSet<RoleFileRevision> RoleFileRevisions { get; set; }
public DbSet<PlaybookGlobalRoleRef> PlaybookGlobalRoleRefs { get; set; }
public DbSet<PlaybookRole> PlaybookRoles { get; set; }
public DbSet<PlaybookRoleFile> PlaybookRoleFiles { get; set; }
public DbSet<PlaybookRoleFileRevision> PlaybookRoleFileRevisions { get; set; }
public DbSet<PlaybookRunTarget> PlaybookRunTargets { get; set; }
```

## Step 2 — OnModelCreating configuration

Add configuration blocks for each new entity. All go in the `vm` schema.

### PlaybookRevision
```csharp
modelBuilder.Entity<PlaybookRevision>(entity =>
{
    entity.ToTable("playbook_revisions", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.PlaybookId).HasColumnName("playbook_id").IsRequired();
    entity.Property(e => e.RevisionNumber).HasColumnName("revision_number").IsRequired();
    entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();
    entity.Property(e => e.ChangeSummary).HasColumnName("change_summary");
    entity.HasOne<Playbook>().WithMany().HasForeignKey(e => e.PlaybookId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.PlaybookId, e.RevisionNumber }).IsUnique();
});
```

### AnsibleRole
```csharp
modelBuilder.Entity<AnsibleRole>(entity =>
{
    entity.ToTable("ansible_roles", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.Name).HasColumnName("name").IsRequired();
    entity.Property(e => e.Description).HasColumnName("description");
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => e.Name).IsUnique();
});
```

### RoleFile
```csharp
modelBuilder.Entity<RoleFile>(entity =>
{
    entity.ToTable("role_files", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.RoleId).HasColumnName("role_id").IsRequired();
    entity.Property(e => e.FilePath).HasColumnName("file_path").IsRequired();
    entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();
    entity.HasOne<AnsibleRole>().WithMany().HasForeignKey(e => e.RoleId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.RoleId, e.FilePath }).IsUnique();
});
```

### RoleFileRevision
```csharp
modelBuilder.Entity<RoleFileRevision>(entity =>
{
    entity.ToTable("role_file_revisions", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.RoleFileId).HasColumnName("role_file_id").IsRequired();
    entity.Property(e => e.RevisionNumber).HasColumnName("revision_number").IsRequired();
    entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();
    entity.Property(e => e.ChangeSummary).HasColumnName("change_summary");
    entity.HasOne<RoleFile>().WithMany().HasForeignKey(e => e.RoleFileId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.RoleFileId, e.RevisionNumber }).IsUnique();
});
```

### PlaybookGlobalRoleRef
```csharp
modelBuilder.Entity<PlaybookGlobalRoleRef>(entity =>
{
    entity.ToTable("playbook_global_role_refs", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.PlaybookId).HasColumnName("playbook_id").IsRequired();
    entity.Property(e => e.AnsibleRoleId).HasColumnName("ansible_role_id").IsRequired();
    entity.HasOne<Playbook>().WithMany().HasForeignKey(e => e.PlaybookId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasOne<AnsibleRole>().WithMany().HasForeignKey(e => e.AnsibleRoleId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.PlaybookId, e.AnsibleRoleId }).IsUnique();
});
```

### PlaybookRole
```csharp
modelBuilder.Entity<PlaybookRole>(entity =>
{
    entity.ToTable("playbook_roles", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.PlaybookId).HasColumnName("playbook_id").IsRequired();
    entity.Property(e => e.Name).HasColumnName("name").IsRequired();
    entity.Property(e => e.Description).HasColumnName("description");
    entity.HasOne<Playbook>().WithMany().HasForeignKey(e => e.PlaybookId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.PlaybookId, e.Name }).IsUnique();
});
```

### PlaybookRoleFile
```csharp
modelBuilder.Entity<PlaybookRoleFile>(entity =>
{
    entity.ToTable("playbook_role_files", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.PlaybookRoleId).HasColumnName("playbook_role_id").IsRequired();
    entity.Property(e => e.FilePath).HasColumnName("file_path").IsRequired();
    entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();
    entity.HasOne<PlaybookRole>().WithMany().HasForeignKey(e => e.PlaybookRoleId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.PlaybookRoleId, e.FilePath }).IsUnique();
});
```

### PlaybookRoleFileRevision
```csharp
modelBuilder.Entity<PlaybookRoleFileRevision>(entity =>
{
    entity.ToTable("playbook_role_file_revisions", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.PlaybookRoleFileId).HasColumnName("playbook_role_file_id").IsRequired();
    entity.Property(e => e.RevisionNumber).HasColumnName("revision_number").IsRequired();
    entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();
    entity.Property(e => e.ChangeSummary).HasColumnName("change_summary");
    entity.HasOne<PlaybookRoleFile>().WithMany().HasForeignKey(e => e.PlaybookRoleFileId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.PlaybookRoleFileId, e.RevisionNumber }).IsUnique();
});
```

### PlaybookRunTarget
```csharp
modelBuilder.Entity<PlaybookRunTarget>(entity =>
{
    entity.ToTable("playbook_run_targets", "vm");
    entity.HasKey(e => e.Id);
    entity.Property(e => e.RunId).HasColumnName("run_id").IsRequired();
    entity.Property(e => e.VmId).HasColumnName("vm_id").IsRequired();
    entity.Property(e => e.AssignmentId).HasColumnName("assignment_id");
    entity.Property(e => e.VarsOverrideSnapshot).HasColumnName("vars_override_snapshot")
        .HasColumnType("jsonb").IsRequired();
    entity.Property(e => e.Status).HasColumnName("status").IsRequired();
    entity.Property(e => e.Stats).HasColumnName("stats").HasColumnType("jsonb").IsRequired();
    entity.Property(e => e.Output).HasColumnName("output").HasColumnType("text");
    entity.HasOne<PlaybookRun>().WithMany().HasForeignKey(e => e.RunId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasOne<VirtualMachine>().WithMany().HasForeignKey(e => e.VmId)
        .OnDelete(DeleteBehavior.Restrict);
    entity.HasIndex(e => e.PublicId).IsUnique();
    entity.HasIndex(e => new { e.RunId, e.VmId });
});
```

## Step 3 — Update existing Playbook and PlaybookRun config

Find the existing `Playbook` entity configuration block and:
- REMOVE property config lines for `GitUrl`, `GitRef`, `PlaybookPath`, `LastRefreshedAt`
- ADD: `entity.Property(e => e.Content).HasColumnName("content").HasColumnType("text").IsRequired();`

Find the existing `PlaybookRun` entity configuration block and:
- REMOVE the `AssignmentId` property config and its FK declaration
- ADD:
  ```csharp
  entity.Property(e => e.PlaybookId).HasColumnName("playbook_id").IsRequired();
  entity.Property(e => e.PlaybookRevisionId).HasColumnName("playbook_revision_id");
  entity.Property(e => e.VarsSnapshot).HasColumnName("vars_snapshot")
      .HasColumnType("jsonb").IsRequired();
  entity.Property(e => e.Output).HasColumnName("output").HasColumnType("text");
  entity.HasOne<Playbook>().WithMany().HasForeignKey(e => e.PlaybookId)
      .OnDelete(DeleteBehavior.Restrict);
  entity.HasOne<PlaybookRevision>().WithMany().HasForeignKey(e => e.PlaybookRevisionId)
      .OnDelete(DeleteBehavior.Restrict);
  ```

## Step 4 — Public ID prefixes

Find `PublicIdInterceptor.cs` (or wherever the entity-type → prefix dictionary is defined).
Add entries for all 9 new entity types:

```csharp
{ typeof(PlaybookRevision),        "pbrev" },
{ typeof(AnsibleRole),             "ansr"  },
{ typeof(RoleFile),                "rf"    },
{ typeof(RoleFileRevision),        "rfrev" },
{ typeof(PlaybookGlobalRoleRef),   "pgr"   },
{ typeof(PlaybookRole),            "plr"   },
{ typeof(PlaybookRoleFile),        "plrf"  },
{ typeof(PlaybookRoleFileRevision),"plrfr" },
{ typeof(PlaybookRunTarget),       "prt"   },
```

## Acceptance Criteria
- `CloudManagerDbContext` compiles with all 9 new DbSets and config blocks
- `Playbook` and `PlaybookRun` config blocks reflect Phase 2 changes
- Public ID prefix map has entries for all 9 new types
- `dotnet build src/CloudManager.API/CloudManager.API.csproj` exits with 0 errors
  (services referencing old git fields will be broken — those are future workflow scope;
  fix only what prevents the build from succeeding in the entity/context layer)

## Test
`Test/0003-ENTITY-BROUHAHA-TEST.md`

## On Failure
- Missing `using` for new entity types in DbContext: add to the using block at the top of the file.
- Build errors in service files referencing `AssignmentId` or git fields: stub or comment out
  only if required to get a clean build — note each one in an Improvise file.
