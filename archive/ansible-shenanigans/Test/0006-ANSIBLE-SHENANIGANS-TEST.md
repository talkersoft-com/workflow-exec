# Test: Python worker + first end-to-end run

## Test ID
`0006-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0006-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After `systemctl start cloud-manager-worker` and the API redeploy. Use a fresh test VM (create one via MCP if needed).

## Test Cases

### TC-001: worker is running
- **Check**: `systemctl is-active cloud-manager-worker`
- **Pass**: `active`
- **Fail**: `journalctl -u cloud-manager-worker -n 50` for cause

### TC-002: health probe sets healthy=true
- **Check**: `psql -c "SELECT enabled, healthy, last_probed_at FROM membership.feature_flags WHERE key='playbooks'"` (or wherever the table landed)
- **Pass**: `healthy=true` and `last_probed_at` within last 6 min
- **Fail**: probe never ran or always fails — check no-op playbook embedded in worker

### TC-003: apply endpoint queues
- **Setup**: flag = (true, true); fresh VM `vm_NEW`; postgres assignment attached
- **Check**: `POST /api/v1/virtualmachine/vm_NEW/playbook/asgn_X/apply`
- **Pass**: 202 with `run_id` starting `run_`; run row exists with status=Queued; `rabbitmqctl list_queues` shows ≥1 message briefly on `playbook-runs`
- **Fail**: 5xx, no queue activity — check publish path

### TC-004: run goes Running → Succeeded
- **Check**: poll `GET /api/v1/run/{run_id}` every 30s
- **Pass**: within ~5 min, status transitions to `Succeeded`; `started_at` and `completed_at` populated; `stats.ok > 0`
- **Fail**: stays Queued (worker not consuming) or goes Failed (read the run's `error_message` + the log file)

### TC-005: log file written
- **Check**: `cat /var/log/cloud-manager/playbook-runs/{run_id}.log | tail -20`
- **Pass**: contains "PLAY RECAP" line with the VM's hostname or IP and counts
- **Fail**: empty file — event_handler not wired

### TC-006: tmp data dir cleaned
- **Check**: `ls /var/lib/cloud-manager/playbook-runs/`
- **Pass**: no entry for completed run_id
- **Fail**: leaked clone — shred + delete didn't fire (could be a try/finally bug)

### TC-007: assignment updated
- **Check**: `GET /api/v1/virtualmachine/vm_NEW/playbook` → the assignment row
- **Pass**: `last_run_id` = the run, `last_run_status` = Succeeded, `last_applied_at` populated
- **Fail**: assignment write missed — race or missing call

## Scoring
All seven. Then tear down the test VM.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
