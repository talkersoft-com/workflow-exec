# Orchestration: ansible-shenanigans

## Workflow ID
`0001-ANSIBLE-SHENANIGANS`

## Objective
Ship generic Ansible playbook execution as a Cloud Manager feature, end-to-end. Start state: no Ansible anywhere, feature flag absent. End state: a user can register a git-hosted playbook, attach it to a VM with typed parameters, click Apply, watch live stdout, and see the playbook's published secrets surfaced from Vault — all behind a `feature_flags.playbooks` flag that flips to enabled+healthy after a passing end-to-end smoke.

Everything playbook-specific lives in the playbook repo behind two YAML conventions (`meta/argument_specs.yml`, `meta/outputs.yml`). Zero playbook-specific code in Cloud Manager.

## Inputs

Read these before starting any task. Every task assumes you have.

- `../init.md` — this workflow's entry point (phase index, cross-phase references, MCP/Vault context)
- `../../planning/PLAYBOOK-INTEGRATION-PLAN.md` — feature plan (phases, data model, API surface, execution flow, secrets strategy, UI)
- `../../planning/ANSIBLE-INSTALL-PLAN.md` — install + Vault policy plan
- `../../planning/DATA-MODEL.md` — current schema
- `../../instructions.md` — workflow conventions (folders, suffixes, /loop, SelfImprove)
- `../../craft-ingot/` — canonical example for every file shape

## Tasks

Run these in order. Each task has a matching `Test/NNNN-*.md`. Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0001-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 0**: Verify install + Vault config (run the pre-built ansible-cli end-to-end against Vault)
- [x] `Tasks/0002-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 1**: Feature flag scaffolding (`feature_flags` table, `/feature-flags` endpoint, `[RequireFeatureFlag]` attribute, web gating)
- [x] `Tasks/0003-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 2**: Playbook data model (`vm.playbooks`, `vm.vm_playbook_assignments`, `vm.playbook_runs` + prefix registry entries)
- [x] `Tasks/0004-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 3**: Playbook registry API (git clone + read `meta/argument_specs.yml` → JSON Schema)
- [x] `Tasks/0005-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 4**: Parameter form web UI (`@rjsf/core` renders argument_schema)
- [x] `Tasks/0006-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 5**: Python worker + RabbitMQ + first end-to-end run
- [x] `Tasks/0007-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 6**: Vault scoped child tokens + outputs surfacing
- [x] `Tasks/0008-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 7**: SignalR run log streaming
- [x] `Tasks/0009-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 8**: Schema diffing (422 with structured diff)
- [x] `Tasks/0010-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 9**: End-to-end smoke against a real postgres playbook
- [x] `Tasks/0011-ANSIBLE-SHENANIGANS-TASK.md` — **Phase 10**: Flip the flag

## Orchestration Steps

1. Read all inputs above before starting Task 0001
2. For each task in order:
   1. Read the task file
   2. Implement the work
   3. Run the matching test file
   4. If the test fails, write `Improvise/0001-ANSIBLE-SHENANIGANS-IMPROVISE-NNN.md` documenting the failure and adaptation, then retry
   5. When the test passes, check the box in this file (edit this orchestration in place)
   6. Move to the next task
3. When every box is checked, write the workflow-level Result and Wishlist (see below) and stop

## Autonomous Execution

```
/loop Continue executing tasks in this orchestration file. For each unchecked task in order: read the task file at the referenced path, do the work, run the matching Test/<n> file, write an Improvise on failure and retry until pass, then edit this orchestration in place to check the box. When every box is checked, write Results/0001-ANSIBLE-SHENANIGANS-RESULT.md (workflow-level summary across all phases) and SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md (workflow-level reflection — MCPs, docs, access that would have compounded across phases) and stop.
```

## Improvisation Policy

- One Improvise file per distinct failure mode encountered, numbered sequentially (`IMPROVISE-001`, `IMPROVISE-002`, ...).
- Reference the task and test case that failed.
- If a prior improvisation covers the same failure, reference it instead of rewriting.
- Do NOT silently retry. Write first, then retry.
- A bug in an earlier phase discovered while running a later one: fix it inline, write Improvise documenting "discovered in Phase N, originated in Phase M, fixed at..." — do NOT walk back and rewrite the earlier phase's task/test files.

## End-of-Workflow Outputs

When every task box is checked:

1. **`Results/0001-ANSIBLE-SHENANIGANS-RESULT.md`** — workflow-level result. Lists which phases needed improvisations, total wall-clock, smoke-VM ID left up for inspection, the final flag state.
2. **`SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md`** — workflow-level reflection. Look across the per-task work; what kept coming up? What MCP / doc / access would have cut total wall-clock the most? Be specific (named MCP method, exact doc path, exact friction). This file informs the operator's next tooling investment — not a complaint log.

## References

- Tasks: `Tasks/0001-...` through `Tasks/0011-...`
- Tests: `Test/0001-...` through `Test/0011-...`
- Improvise: `Improvise/` (create on failure)
- Result: `Results/0001-ANSIBLE-SHENANIGANS-RESULT.md` (final)
- Wishlist: `SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md` (final)
