# Task: End-to-end smoke (postgres)

## Task ID
`0010-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Prove the whole stack works against a real workload.

## Steps

1. **Flag on**: `PATCH /api/v1/admin/feature-flags/playbooks {enabled: true}` (worker probe should already have set healthy=true).

2. **Create test VM via MCP**: `vm_create(host_KW7YKFF3YM, "smoke-postgres", 4096, vmi_2QMK0FHWXG)`. Wait for status 2 (Completed) and machineStatus 0 (Running).

3. **Register playbook**: pick one of
   - `geerlingguy.postgresql` (mature, has argument_specs)
   - a small custom playbook in a Talkersoft repo that includes `meta/outputs.yml` writing `admin_password`, `connection_uri`, `port` to `{{ secrets_prefix }}/postgres/`
   The custom playbook is preferred — it's the one that actually exercises Phase 0007's outputs surfacing.

4. **Attach**: POST attach with `vars_override` overriding the default postgres version and adding one database.

5. **Apply**: click Apply in the web UI. Open the run page, watch live log.

6. **Validate every layer**:
   - **DB**: `playbook_runs.status = Succeeded`, `stats.failures = 0`, `secrets_published` has 3+ entries
   - **Vault**: `vault kv get cloudmanager/vm/instances/{vm_pub}/{pb_pub}/postgres/admin_password` returns a value
   - **UI**: secrets table shows the three entries; Reveal works for `admin_password`
   - **VM**: SSH into the VM with the cached private key, run `sudo -u postgres psql -c "\l"`, confirm the database from `vars_override` exists
   - **VM**: confirm postgres listens on the configured port via `ss -tlnp | grep 5432`
   - **End-to-end**: extract `connection_uri` from Vault → run `psql "$URI" -c "SELECT version()"` from the cloud-manager host → returns a version string

## Acceptance Criteria
All six validations above pass. Live log streamed during the run (Phase 0008 exercised). Run took less than 10 min wall-clock.

## Test
`Test/0010-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
This phase is allowed to surface bugs in prior phases. Document inline: write Improvise, fix the prior phase's code, redeploy, re-run from here. Do NOT walk back and "fix the docs" of the prior phase — the workflow record is a timeline.

## Cleanup
**Do not destroy the VM.** The operator will inspect via the UI. They can call `orchestration_destroy` themselves when finished.
