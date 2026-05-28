# Task: Python worker + first end-to-end run

## Task ID
`0006-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
First successful playbook execution end-to-end: web Apply → API → RabbitMQ → worker → ansible-runner → real ssh into a real VM → ansible runs to completion → run row marked Succeeded with stdout in a log file.

## Context
- Worker placement: per the plan's open question, **separate `cloud-manager-worker` service**. Keep it in a new directory under `cloud-manager-api/cloud-manager-worker/` for now (single repo, multiple deployables). Reuse the existing `scripts/api/install-api-service.py` shape for its own install script.
- Run as `cloudmanager` user. EnvironmentFile holds `VAULT_TOKEN` (long-lived orchestrator token). NOT in YAML — that's the tech debt already flagged on vorch-service.
- Health probe: simple — every 5 min, invoke `ansible_runner.run` against a no-op playbook on localhost; on success, `UPDATE feature_flags SET healthy=true, last_probed_at=NOW() WHERE key='playbooks'`.

## Steps

1. **Worker scaffolding**
   - `cloud-manager-worker/` dir with `cloud-manager-worker` bash wrapper, `requirements.txt` (`pika`, `ansible-runner`, `psycopg`, `python-vault` or `hvac`), `cli/main.py`.
   - `pipx`-style venv at `~/.local/pipx/venvs/cloud-manager-worker/` for the cloudmanager user (mirror Phase 0001 install pattern).

2. **Consumer**
   - `pika.BlockingConnection` to localhost RabbitMQ, declare `playbook-runs` queue, basic_consume with manual ack.
   - On message: dispatch to `RunHandler(run_id).execute()`. Catch exceptions, mark run Failed in DB, ack so it doesn't requeue forever.

3. **`RunHandler.execute`**
   1. Read run row + assignment + playbook + VM + host + network address from Postgres.
   2. Set run.status = Running, run.started_at = NOW.
   3. Build `secrets_prefix = cloudmanager/vm/instances/{vm_pub}/{pb_pub}`. Store on the run row.
   4. Pull VM SSH key from Vault path `cloudmanager/vm/instances/{vm_pub}` via hvac. Write to tmpfile `0600`.
   5. Shallow git clone the playbook to a tmpdir on `/var/lib/cloud-manager/playbook-runs/{run_id}/project/`.
   6. Build runner private_data_dir: `inventory/hosts.yml` (one host, the VM), `env/extravars` (vars_override + secrets_prefix + vm_public_id + playbook_public_id), `env/envvars` (VAULT_ADDR), `env/ssh_key` (the tmpfile).
   7. `ansible_runner.run(private_data_dir=..., playbook=playbook.playbook_path, quiet=True)`.
   8. Capture `r.rc`, `r.stats`. Tee stdout to `/var/log/cloud-manager/playbook-runs/{run_id}.log` via the runner's `event_handler`.
   9. Update run row: status = Succeeded/Failed, completed_at, stats, error_message if any.
   10. Update assignment row: last_run_id, last_run_status, last_applied_at.
   11. Delete the private_data_dir (shred the ssh_key first).

4. **API apply endpoint**
   - `POST /api/v1/virtualmachine/{vmId}/playbook/{asgnId}/apply` — inserts run row (status=Queued), publishes `{run_id}` to `playbook-runs` queue, returns 202 with `{run_id}`.

5. **Run read endpoint**
   - `GET /api/v1/run/{runId}` returns the row + `assignment_id → assignment → playbook` joined.
   - `GET /api/v1/run/{runId}/log` returns the contents of the log file (full, not paginated yet).

6. **Web wiring**
   - Apply button → `api.assignments.apply(vmId, asgnId)` → toast with run_id → poll `/run/{runId}` every 2s until terminal status → refresh assignments.

7. **Health probe**
   - Background thread in the worker. Every 5 min: `ansible_runner.run` against an embedded no-op playbook. On success, update `feature_flags.playbooks.healthy=true`. On failure, set false + log.
   - Also fire once at worker startup so a restart doesn't leave `healthy=false` for 5 min.

8. **Install + start services**
   - `cloud-manager-worker/scripts/install.py` (mirror `scripts/api/install-api-service.py`).
   - Systemd unit: `cloud-manager-worker.service`. Enable + start.

## Acceptance Criteria
- A fresh test VM, with the postgres assignment from Phase 0005, runs Apply → 5 minutes later (postgres install is slow) the run row is `Succeeded` and the log file contains the ansible play recap with non-zero `ok` counts.
- The worker survives RabbitMQ restart (auto-reconnect).
- Health probe runs at startup; flag flips to healthy=true within 1 minute of `systemctl start cloud-manager-worker`.

## Test
`Test/0006-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
The SSH key fetch + tmpfile dance is fragile. Most common failure: VM IP changed and the SSH attempt hangs. ansible-runner's default 10s connection timeout is fine but watch for it.
