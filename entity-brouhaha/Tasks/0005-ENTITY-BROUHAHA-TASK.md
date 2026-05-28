# Task: Generate EF migration + apply to local and remote DB

## Task ID
`0005-ENTITY-BROUHAHA-TASK`

## Repo
`cloud-manager-api` (on branch `entity-brouhaha`)

## Objective
Generate the EF Core migration that captures all schema changes from Phases 1–4, apply it to the
local database, then apply it to the remote database via the SSH tunnel.

## Step 1 — Generate the migration

```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
python3 scripts/database/add-migration.py AddPlaybookManagement
```

**Review the generated `Up()` before applying.** Verify it contains:
- `CreateTable` for all 9 new tables in the `vm` schema
- `DropColumn` for `git_url`, `git_ref`, `playbook_path`, `last_refreshed_at` on `vm.playbooks`
- `AddColumn` for `content` on `vm.playbooks`
- `DropColumn` for `assignment_id` on `vm.playbook_runs`
- `AddColumn` for `playbook_id`, `playbook_revision_id`, `vars_snapshot`, `output` on `vm.playbook_runs`
- FK constraints and indexes for all new tables

If anything is missing, investigate the DbContext config before applying.

## Step 2 — Handle existing data

Check whether `vm.playbook_runs` has any rows:
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5433 -U postgres -d clouddb \
  -tc "SELECT count(*) FROM vm.playbook_runs;"
```

If **0 rows**: proceed directly to Step 3. The `AddColumn` for non-nullable `playbook_id` will use
EF's default value (empty GUID) — acceptable for an empty table.

If **rows exist**: add a backfill in the generated `Up()` after structural changes:
```csharp
migrationBuilder.Sql(@"
    UPDATE vm.playbook_runs pr
    SET playbook_id = pa.playbook_id
    FROM vm.vm_playbook_assignments pa
    WHERE pr.playbook_id = '00000000-0000-0000-0000-000000000000'
");
migrationBuilder.Sql("UPDATE vm.playbook_runs SET vars_snapshot = '{}' WHERE vars_snapshot IS NULL;");
```

Document the decision in an Improvise file either way.

## Step 3 — Apply to local DB (port 5433)

```bash
python3 scripts/database/deploy-database.py
```

Confirm it targets the local DB. If the script targets port 5432 (remote tunnel) instead, check
the script's config and run against local first before touching remote.

## Step 4 — Apply to remote DB (port 5432 via SSH tunnel)

The SSH tunnel must be active on port 5432 (`ubuntu-server.talkersoft.com`). Verify:
```bash
nc -zv localhost 5432 && echo "tunnel up" || echo "tunnel down"
```

If the deploy script reads from `database.json` and targets `clouddb` (port 5432), run it again
with the appropriate flag or connection override to hit the remote. If the script handles both
databases in sequence, confirm both are applied.

## Acceptance Criteria
- Migration file `AddPlaybookManagement` exists in `Migrations/`
- Local DB (port 5433): all 9 new tables exist in `vm` schema
- Remote DB (port 5432 via tunnel): all 9 new tables exist in `vm` schema
- `vm.playbooks` has `content`, no git columns
- `vm.playbook_runs` has `playbook_id`, no `assignment_id`

## Test
`Test/0005-ENTITY-BROUHAHA-TEST.md`

## On Failure
- Migration name collision: use `AddPlaybookManagementV2`
- Non-nullable `playbook_id` add-column fails on existing rows: add `.HasDefaultValue(Guid.Empty)` temporarily to the DbContext property, regenerate, then use SQL to backfill and drop the default
- Tunnel not up: start the tunnel (`ssh -fNL 5432:localhost:5432 todd@ubuntu-server.talkersoft.com`) then retry Step 4
- FK violation on `assignment_id` drop: check `\d+ vm.playbook_runs` in psql for unexpected constraints referencing it
