# Execution: cloud-manager/ansible-p2-decomposition

## Objective
Stored playbooks are decomposed into addressable records — PlaybookPlay (`play`), PlaybookTask
(`ptask`), PlaybookHandler (`hdlr`), VarBlock (`vblk`) — rebuilt atomically per revision by a
YamlDotNet decomposer service hooked into every playbook/role-file save, with a typed
EntityEdge (`edge`) graph (`depends_on`, `includes`, `consumed_by`, `targets`, `defines`,
`overrides`), cycle detection on role dependencies, flag-gated usedBy/graph/read APIs, an
idempotent decompose-all backfill, and MCP tools in a new `graph.ts` module. The YAML blob
stays the source of truth — records are a derived read model, never recomposed to YAML
(structured-edit PATCH is explicitly deferred to after P3). Existing playbook/role/run flows
are byte-identical. API + MCP built, migration applied, deployed, and shipped on one
exec-stage PR set.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/ansible-p2-decomposition/PLAN.md`
- `../../../workflow-plans/cloud-manager/ansible-p1-inventory/PLAN.md`
- `../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Part A record entities (`play` `ptask` `hdlr` `vblk`) + EF configs + migration
- [x] `Tasks/0002-TASK.md` — **Phase 2**: Part A decomposer — YamlDotNet `IPlaybookDecomposer`, transactional rebuild, failure marking, save-path hooks
- [x] `Tasks/0003-TASK.md` — **Phase 3**: Part B edge graph — `edge` entity + migration; edge writes from decomposer; role-dep cycle check (400)
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Part B APIs + backfill — usedBy/graph/read endpoints, decompose-all, failure events
- [x] `Tasks/0005-TASK.md` — **Phase 5**: MCP `graph.ts` module + profile registration + regression (existing flows byte-identical) + backfill against seed data
- [x] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api + mcp → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p2-decomposition"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
