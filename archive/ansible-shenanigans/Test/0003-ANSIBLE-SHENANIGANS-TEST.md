# Test: Playbook data model

## Test ID
`0003-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0003-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After `dotnet ef database update` (or equivalent) completes.

## Test Cases

### TC-001: tables exist in vm schema
- **Check**: `psql -c "\dt vm.*"`
- **Pass**: `vm.playbooks`, `vm.vm_playbook_assignments`, `vm.playbook_runs` all listed
- **Fail**: missing table → migration not applied

### TC-002: playbooks columns + types
- **Check**: `psql -c "\d vm.playbooks"`
- **Pass**: contains `name (varchar 255 unique)`, `git_url (varchar 2048)`, `git_ref (varchar 255)`, `playbook_path (varchar 255)`, `argument_schema (jsonb)`, `output_schema (jsonb)`, `last_refreshed_at (timestamptz)`, plus all audit columns
- **Fail**: type drift from plan → fix entity, re-migrate

### TC-003: assignments unique constraint
- **Check**: `psql -c "\d vm.vm_playbook_assignments" | grep -i unique`
- **Pass**: unique index on `(virtual_machine_id, playbook_id)`
- **Fail**: dup assignments would be allowed — add the constraint

### TC-004: runs FK to assignments
- **Check**: `psql -c "\d vm.playbook_runs" | grep -i 'FK\|REFERENCES'`
- **Pass**: `assignment_id` references `vm.vm_playbook_assignments(id)`
- **Fail**: FK missing → orphan runs possible

### TC-005: enum has all six values
- **Check**: `psql -c "SELECT unnest(enum_range(NULL::playbook_run_status))"`
- **Pass**: `None, Queued, Running, Succeeded, Failed, Cancelled`
- **Fail**: missing value → rebuild enum migration

### TC-006: prefix registry covers new entities
- **Check**: `grep -E '"pb"|"asgn"|"run"' cloud-manager-api/src/Models/CloudManager.Entities/PublicId/EntityPrefixRegistry.cs`
- **Pass**: all three present
- **Fail**: insert row → trigger fires → public_id generation would throw

### TC-007: insert smoke
- **Check**: insert a dummy `vm.playbooks` row via psql with minimal columns; confirm `public_id` is auto-populated with `pb_<10 char Crockford>`
- **Pass**: row exists, public_id matches `^pb_[0-9A-Z]{10}$`
- **Fail**: trigger / interceptor not firing for new entity type
- **Cleanup**: delete the row before continuing

## Scoring
All seven. Cleanup is mandatory.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
