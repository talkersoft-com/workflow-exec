# Execution: cloud-manager/ansible-p3-roundtrip

## Objective
A ruamel.yaml round-trip engine runs as a Python sidecar (`roundtrip-service/`, FastAPI +
ruamel, own Dockerfile) beside the API: `POST /decompose` returns a structural map with
per-node source spans + style metadata, `POST /recompose` splices structured edits back into
the original text with byte-identical formatting (comments, key order, anchors, block-scalar
styles preserved). A `ydoc` record per `PlaybookRevision`/`RoleFileRevision` stores the source
checksum + decomposition map and a write-time `recompose(decompose(x)) == x` check fails
loudly on mismatch. Structured-edit PATCH endpoints (play/task) build an edit list, splice via
the sidecar, and save through the EXISTING revision-cutting path so history, events, and P2
re-decomposition fire unchanged. P2's decomposer prefers the sidecar span map when present.
Golden-file fidelity corpus green, raw-text save path byte-identical, API built, migration
applied, deployed, and shipped on one exec-stage PR set. No porch/vorch/mcp/web changes.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p3-roundtrip/PLAN.md`
- `../../../../workflow-plans/cloud-manager/ansible-p2-decomposition/PLAN.md`
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §1 (`ydoc`), §3 round-trip routes
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §2 (round-trip fidelity), §11 (data integrity)

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: sidecar service — FastAPI + ruamel decompose/recompose/healthz, Dockerfile, golden-file corpus
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: `ydoc` entity + migration; API proxy routes `api/v1/roundtrip/*`; write-time fidelity check on revision cut
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: structured edits — PATCH play/task endpoints → edit list → splice → existing revision path
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: P2 integration — decomposer prefers sidecar span map; spans flow into `ptask.SourceSpan`
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: regression + docs — raw-text save path byte-identical; corpus green; deployment docs
- [ ] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p3-roundtrip"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
