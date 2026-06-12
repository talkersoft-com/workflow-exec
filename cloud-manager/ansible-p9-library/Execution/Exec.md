# Execution: cloud-manager/ansible-p9-library

## Objective
The epic's data layer becomes the reuse-first UX it exists for. **Part A**: `AnsibleRole` gains
additive library metadata (`Tags`, `SupportedPlatforms`, `Maintainer`, `Version` + migration,
DTOs, PATCH support, list filters `?tag=&platform=&q=` — the plan's only non-web diff); a
library browser at `/ansible/library` unifies search across roles/playbooks/collections with
tag chips, platform badges, and inline used-by counts from C-GRAPH; a dependency graph view
renders `GET api/v1/entity/{id}/graph?depth=2` as an interactive SVG (d3-force for layout math
only, custom React rendering, depth/edge-type filters, click-through). **Part B**: a composer
at `/ansible/compose` where picked roles form an ordered list and each role's C-VAR interface
renders as an rjsf form — `required` prominent, `default` collapsed with override affordance,
`internal` hidden, secret-classified params through C-DECO's `<SecretValue>`/ref-picker writing
`$secretRef` only — validated before save via C-RES dry resolve (every required param satisfied
or explicitly deferred); a per-target effective-config preview pane (P8's context bar reused)
listing resolved vars + provenance + gaps, gaps blocking when `enforceRequired` is on;
guardrails — used-by warning modals on edit/delete of shared entities ("used by N playbooks /
M tasks" with the C-GRAPH list, `ConfirmModal` danger pattern), breaking-change list when a
required var of a consumed role is removed/renamed, lint-gate toggle blocking runs while head
revision has `plint` errors; and `RunDetailPage` upgraded to tail-follow
`GET …/log/chunk?afterSeq=` (2s poll), per-task grouping, check-mode runs labelled
"dry-run — would change". Existing `/ansible/*` and blueprint/marketplace pages byte-identical.
API + web built, migration applied, both deployed; shipped on one exec-stage PR set.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p9-library/PLAN.md` — the design, the A/B split, the open-question defaults
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §5 C-GRAPH, §7 C-VAR, §8 C-RES (all consumed verbatim, never reshaped)
- `../../../../workflow-plans/cloud-manager/ansible-p2-decomposition/PLAN.md`, `../../../../workflow-plans/cloud-manager/ansible-p5-resolution/PLAN.md`, `../../../../workflow-plans/cloud-manager/ansible-p8-decorations/PLAN.md` — the dependency surface (used-by APIs, preview, C-DECO components)
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §3 (library objects, impact-aware editing), §9 (browser, composer, preview, console), §10 (guardrails)

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; verify P2/P5/P8 merged; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1 (Part A)**: role library metadata — additive `ansible_roles` fields + EF migration, DTOs, PATCH, list filters (only non-web diff)
- [ ] `Tasks/0002-TASK.md` — **Phase 2 (Part A)**: library browser + dependency graph — `/ansible/library`, unified search, tag/platform facets, used-by counts, graph view component
- [ ] `Tasks/0003-TASK.md` — **Phase 3 (Part B)**: composer — role picking + ordering, C-VAR interface forms (rjsf), secret ref-picker, validate-before-save via C-RES dry resolve
- [ ] `Tasks/0004-TASK.md` — **Phase 4 (Part B)**: effective-config preview pane + guardrails — used-by warning modals, breaking-change check, lint-gate toggle
- [ ] `Tasks/0005-TASK.md` — **Phase 5 (Part B)**: run console tail-follow + check-mode labelling; regression — existing pages byte-identical; e2e composer→preview→run
- [ ] `Tasks/0006-TASK.md` — **Final phase**: apply migration → deploy api + web → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p9-library"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
