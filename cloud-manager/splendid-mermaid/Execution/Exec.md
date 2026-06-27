# Execution: cloud-manager/splendid-mermaid

## Objective
A blueprint with a templated `{vm_public_id}` secret binding no longer bricks provisioning.
Secret bindings resolve BEFORE the VM record exists, so any failure returns a clean 422 (binding
name + Vault path, never the value) with no orphaned VM. `InstantiateRequest.secretBindingParams`
lets an operator supply the source VM for `{vm_public_id}` so a consumer VM is injected with another
VM's existing creds; a missing required param is a 422, never a brick. The MCP
`cloud_marketplace_provision` round-trips the new param. No-binding, static, and concrete-path
templated bindings stay identical. No DB migration; vorch unchanged. Verified end-to-end on the
deployed system.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/splendid-mermaid/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: API P0 — resolve-before-create + clean 422 + failure-event guard
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: API P1 — `secretBindingParams`; resolver uses supplied params; required-missing → 422
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: MCP — `cloud_marketplace_provision` optional `secretBindingParams` passthrough
- [ ] `Tasks/0004-TASK.md` — **Final phase**: E2E + build api+mcp + deploy api; write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "splendid-mermaid"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- Two failed recoveries on the same failure → stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
