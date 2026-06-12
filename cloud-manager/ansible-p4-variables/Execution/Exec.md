# Execution: cloud-manager/ansible-p4-variables

## Objective
Variables become first-class typed registry records in `cloud-manager-api`: every variable a
role, playbook, or inventory declares is a `VariableDefinition` (`vdef`) with a kind
(`required | default | internal`) and a sensitivity (`plain | secret`), evented append-only
(`varev`). Secret variables never carry values — they carry a `SecretRef` (`sref`) pointing
into Vault, and every override surface (`VmPlaybookAssignment.VarsOverride`,
`BlueprintPlaybook.VarsOverride`, P1 `gvar`/`hvar` values) accepts the
`{ "$secretRef": "sref_…" }` form while cleartext writes to secret-classified names are
rejected (409, DB CHECK + service validation). At materialization, `$secretRef` values expand
into the EXISTING `MaterializedPlaybook.SecretInputs` entries — porch resolves them exactly as
today and does not change. The registry is populated by CRUD (`api/v1/variable`, gated
`ansible-studio`), by harvest from P2's decomposition output, and a migration re-keys P2's
`var:` pseudo-id edges to `vdef_…` ids where names match. Required variables become
enforceable: `GET api/v1/playbook/{pid}/requiredVars` reports the missing list over the role
closure, and the run trigger gains an additive `enforceRequired` flag (default false). C-VAR
(contracts §7) is exposed verbatim — P5 and P8 consume it. `variables.ts` MCP tools mirror the
registry. Existing runs with untyped vars byte-identical (incl. blueprint sequencer),
migration applied, api + mcp built and deployed, shipped on one exec-stage PR set. No
web/vorch/porch changes.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p4-variables/PLAN.md`
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §1 (`vdef` `sref` `varev`), §4 (MCP), §6 (event types), §7 C-VAR (exposed verbatim — binding), §11 invariants
- `../../../../workflow-plans/cloud-manager/ansible-p1-inventory/PLAN.md` — `gvar`/`hvar` layers this plan types
- `../../../../workflow-plans/cloud-manager/ansible-p2-decomposition/PLAN.md` — `vblk` blocks, `defines`/`consumed_by` edges this plan harvests and re-keys
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §3, §4, §11

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: entities + migration — `vdef`/`sref`/`varev`, CHECK constraints, P1 var-table nullable `variable_definition_id` retrofit, P2 edge re-key
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: registry API — CRUD + secretRef routes, events, harvest endpoint
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: typed overrides — `$secretRef` validation on every override write path; materialization expands refs into SecretInputs
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: required enforcement — requiredVars endpoint over the role closure; optional `enforceRequired` on run trigger
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: MCP + regression — `variables.ts`; cleartext-rejection tests; untyped-var run flows byte-identical
- [ ] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api + mcp → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p4-variables"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
