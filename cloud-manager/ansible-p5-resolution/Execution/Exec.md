# Execution: cloud-manager/ansible-p5-resolution

## Objective
A precedence-aware variable resolution engine lives in `cloud-manager-api` as a pure,
unit-tested service: given a concrete operation context (playbook + inventory + host +
optional extra-vars) it folds every record layer (role defaults → inventory group vars by
nesting depth → host vars → play vars → assignment override → blueprint override → extra
vars) and reports, per variable, the effective value, the winning layer with source public
id, the overridden chain, and a state (`resolved | undefined | missingRequired`). Secret
masking (`"***"`) is enforced inside the engine — no caller can receive cleartext. C-RES
(contracts §8) is exposed verbatim via `POST api/v1/resolution/preview` (gated
`ansible-studio`, per-host matrix mode when `hostId` is null, 200-host cap → 413), a `rsnap`
ResolutionSnapshot is captured per run target at enqueue and readable via
`GET api/v1/run/{runId}/resolution`, `enforceRequired: true` on the run trigger returns 422
with the gap list, and the `cloud_resolution_preview` MCP tool mirrors C-RES. Synthetic-scale
perf test green, existing run flows byte-identical with the flag off, migration applied, api
+ mcp built and deployed, shipped on one exec-stage PR set. No web/vorch/porch changes.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p5-resolution/PLAN.md`
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §1 (`rsnap`), §4 (MCP), §7 C-VAR (consumed verbatim), §8 C-RES (exposed verbatim — binding)
- `../../../../workflow-plans/cloud-manager/ansible-p1-inventory/PLAN.md` — inventory layers (`gvar`/`hvar`, group nesting)
- `../../../../workflow-plans/cloud-manager/ansible-p2-decomposition/PLAN.md` — `vblk` variable blocks
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §5, §6, §10, §11

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: precedence engine — layer descriptors as data, fold algorithm, engine-level masking, unit suite incl. C-VAR fixtures
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: preview endpoint — C-RES DTOs verbatim, `POST api/v1/resolution/preview`, `ansible-studio` gating, per-host matrix mode
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: run snapshot — `rsnap` entity + migration, capture at enqueue, `GET api/v1/run/{runId}/resolution`
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: gap wiring + MCP — `enforceRequired` 422 path, `resolution.ts` tool, additive gap field in run-list DTOs
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: perf + regression — synthetic 1k-var / 200-host perf test; flag-off run flows byte-identical
- [ ] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api + mcp → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p5-resolution"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
