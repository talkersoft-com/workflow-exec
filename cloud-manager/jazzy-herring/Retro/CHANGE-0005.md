# CHANGE-0005 — Assignment tracking + log redaction parity

## Assignment tracking — added in cloud-manager-api
Per CHANGE-0002 finding (the Python worker was the sole writer of `vm_playbook_assignments.LastRunId/Status/AppliedAt`), the writeback is now in `cloud-manager-api/src/Services/CloudManager.Data.Services/PlaybookRunTargetService.cs::UpdateAsync`:

```csharp
if (isTerminal && entity.AssignmentId.HasValue)
{
    var assignment = await _context.VmPlaybookAssignments
        .FirstOrDefaultAsync(a => a.Id == entity.AssignmentId.Value);
    if (assignment is not null)
    {
        assignment.LastRunId = run.Id;
        assignment.LastRunStatus = entity.Status;
        assignment.LastAppliedAt = DateTime.UtcNow;
        await _context.SaveChangesAsync();
    }
}
```

Per-target write semantics (each target updates its own assignment, using the target's own status) — matches what the Python worker did when it processed runs that were 1 target = 1 assignment. Multi-VM runs now correctly write back to each assignment independently.

Sits BEFORE the existing run-rollup block in the same method, so it runs even if not all sibling targets are terminal yet. The run-rollup block (which sets `run.Status` + revokes Vault) is unchanged.

## Log redaction — already wired in cmd/porch/main.go (Phase 3 CHANGE-0003)
`cmd/porch/main.go` registers a `redactingWriter` around `os.Stdout` (and `/kvm-automator/porch.log` when running as service). The pattern `hvs\.[A-Za-z0-9_\-]+` is replaced with `hvs.<REDACTED>` before bytes hit either sink. Since `log.SetOutput` is called in `init()`, every internal package that uses the stdlib `log` is automatically redacted — no per-package opt-in required.

## Token-leak audit of ansible handlers
```
grep -nE 'log\.(Printf|Println|Print).*(VaultRunToken|child_token|childToken|VAULT_TOKEN.*=)' internal/ansible/*.go
→ zero matches
```

The two existing log lines that mention "token" both log errors (`%v` on `err`) without the token value:
```
internal/ansible/playbook_run_exec.go:194: log.Printf("playbook-run: target %s mint child token: %v", target.ID, err)
internal/ansible/playbook_run_exec.go:201: log.Printf("playbook-run: target %s revoke child token: %v", target.ID, err)
```

Even if a logical error path did include the token, the porch-side redactingWriter would scrub it.

## Build status
- `cloud-manager-api` `dotnet build`: 0 errors, 9 warnings (pre-existing, none related to this change).
- `vorch-service` `go build ./...`: exit 0.
