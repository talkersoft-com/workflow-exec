# Improvise: Backfill required — existing playbook_runs row

## Task
`0005-ENTITY-BROUHAHA-TASK.md` — Step 2 (Handle existing data)

## Situation
Local `vm.playbook_runs` contained 1 existing row:
- `id`: `d6bfab84-708e-4c4c-922d-ea1de2c3b7a5`
- `assignment_id`: `3ad6688d-a12b-4d8f-a039-a51df6fd4873`

EF Core generated `RenameColumn(assignment_id → playbook_id)` rather than DropColumn + AddColumn,
because both columns are `Guid NOT NULL` and EF detected the pattern as a rename. After the rename,
`playbook_id` contains the old assignment GUID `3ad6688d-...`, not a real playbook GUID. The new FK
`FK_playbook_runs_playbooks_playbook_id → vm.playbooks.id` would fail on this row.

## Action Taken
Added two `migrationBuilder.Sql` calls immediately before the `AddForeignKey` calls:

1. **Backfill playbook_id** — joins `playbook_runs.playbook_id` (containing the old assignment GUID)
   back to `vm_playbook_assignments.id` to read the real `playbook_id`:
   ```sql
   UPDATE vm.playbook_runs pr
   SET playbook_id = pa.playbook_id
   FROM vm.vm_playbook_assignments pa
   WHERE pr.playbook_id = pa.id
   ```
   Verified: assignment `3ad6688d-...` → playbook `40f62c9a-f539-4922-9aa9-67d80149396b`.

2. **Fix vars_snapshot default** — EF used `defaultValue: ""` for the new JSONB column, which
   produces an empty string for existing rows. Updated to `'{}'`:
   ```sql
   UPDATE vm.playbook_runs SET vars_snapshot = '{}' WHERE vars_snapshot = '' OR vars_snapshot IS NULL;
   ```
