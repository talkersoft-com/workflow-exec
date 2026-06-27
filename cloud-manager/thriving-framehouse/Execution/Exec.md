# Orchestration: cloud-manager/thriving-framehouse

## Objective
Done looks like: a templated secret binding can be provisioned **through the MCP**
(`cloud_marketplace_provision` with `secretBindingParams:[{secretBindingId, params:{vm_public_id:…}}]`)
without a camelCase guard error, and `GET /api/v1/vm/{publicId}/secret-binding` (and
`cloud_vm_secret_binding_list`) returns the **true** resolved Vault path of the source VM — read
from a persisted `resolved_vault_path` column rather than recomputed against the listing VM's id.
A single reversible migration adds the column; old rows fall back to today's recompute. Both
services build clean and `cloud-manager-api` is deployed before integrate.

## Inputs
Read these before Task 0000:
- `../init.md` — objective, scope, conventions, backward-compat.
- `../deck.md` — deck operations, branch, affected repos.
- `/home/todd/workspace/cloud-manager/planning/workflow-plans/cloud-manager/thriving-framehouse/PLAN.md` — full design.

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1 (API)**: reversible migration adding `resolved_vault_path` to `vm.vm_secret_bindings`; populate from `SecretBindingResolver` at provision; read endpoint returns stored value, recompute only when null; build api
- [x] `Tasks/0002-TASK.md` — **Phase 2 (MCP)**: exempt `secretBindingParams[].params` map keys from the camelCase guard; guard unchanged elsewhere; build mcp
- [x] `Tasks/0003-TASK.md` — **Final phase**: E2E through MCP, apply migration, build api+mcp, deploy api, write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000.
2. For each unchecked task in order:
   a. Read the task file.
   b. Do the work.
   c. Run the matching Test file (`Test/NNNN-TEST.md`).
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test.
   e. On pass: check the box, move to the next task.
3. When every box is checked, the workflow is complete.

## Autonomous execution
Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "thriving-framehouse"
```
Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...).
- Never silently retry — write the FIX file first, then apply the fix.
- If a failure cannot be recovered after two attempts: stop and surface to operator.

## Build requirements
- `cloud-manager-api` changed → `python3 .cicd/build-cloud-manager.py --target api`
- `cloud-manager-mcp` changed → `python3 .cicd/build-cloud-manager.py --target mcp`
- both → `python3 .cicd/build-cloud-manager.py --target all`
A task is not done until its build passes cleanly.

## Deploy procedure (after all tasks pass, BEFORE hv_integrate)
1. Apply the migration to the live DB **before** restarting the API:
   ```
   cd /home/todd/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities && \
     dotnet ef database update \
       --connection "Host=localhost;User ID=cloudmanager;Password=P@ssw0rd!;Port=5432;Database=clouddb;SslMode=Prefer;Trust Server Certificate=true"
   ```
2. Deploy: `python3 .cicd/deploy-cloud-manager.py --target api` (mcp dist is rebuilt by its build;
   the running MCP server is NOT restarted — operator must `/mcp` reload).
3. A failed deploy is a stop condition — write a FIX file under `Retro/`, resolve, retry.

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
