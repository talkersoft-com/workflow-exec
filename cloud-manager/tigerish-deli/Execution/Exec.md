# Orchestration: cloud-manager/tigerish-deli

## Objective
RabbitMQ is installable from the Cloud Manager marketplace as `rabbitmq-jammy` and
`rabbitmq-noble`. Each instantiation provisions a VM and runs an Ansible playbook that
installs RabbitMQ 4.x (official Team RabbitMQ repos), enables the management plugin,
creates an admin user + vhost, writes the password to Vault, and opens 5672/15672.
`GET /api/v1/marketplace` lists both blueprints; instantiation produces a reachable broker.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/tigerish-deli/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: RabbitMQ playbooks (jammy + noble) + meta.json
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: EF migration seeding both blueprints + links
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Verify (migrate + import + marketplace + instantiate), then write Results + Retro/LESSONS, deploy, hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "tigerish-deli"
```

## Build requirements
- `cloud-manager-api` (.NET) changed → `python3 .cicd/build-cloud-manager.py --target api`
- A task is not done until its build passes cleanly.

## Deploy procedure (after all phases pass, BEFORE hv_ship)
1. Apply the EF migration to the live DB FIRST (see init.md / PLAN.md command).
2. `python3 .cicd/deploy-cloud-manager.py --target api`
3. Run `import-playbooks.py` so `rabbitmq-*` land in `vm.playbooks`.
4. Surface deploy target + smoke-test results in `Results/RESULT.md`.

## Improvisation policy
- One FIX file per distinct failure (FIX-001, FIX-002, …); write it before retrying.
- Two failed recovery attempts on one failure → stop and surface to operator.

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
