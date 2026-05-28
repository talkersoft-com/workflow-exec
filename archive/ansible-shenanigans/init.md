# ansible-shenanigans — Entry Point

You are an agent. This workflow ships generic Ansible playbook execution as a Cloud Manager feature — from "no Ansible anywhere" to "a user can attach a postgres playbook to a VM, click Apply, and see the generated admin password surfaced from Vault."

Read this whole file before anything else. Then read the source plans (see "Required Reading" below). Then start at Phase 0.

## What This Workflow Accomplishes

End state: a user can register a git-hosted Ansible playbook with Cloud Manager, attach it to a VM with typed parameters, click Apply, watch live stdout, and see the playbook's published secrets surfaced from Vault. Zero playbook-specific code in Cloud Manager — everything playbook-specific lives in the playbook repo, behind two YAML conventions (`meta/argument_specs.yml`, `meta/outputs.yml`).

The feature ships behind a `feature_flags.playbooks` flag and stays invisible (404 on routes) until the flag flips to enabled+healthy.

## Required Reading

Before starting, read these in order. Do not skip — every Phase references them.

1. `../../planning/ANSIBLE-INSTALL-PLAN.md` — the install + Vault policy plan. **Note:** the `cloud-manager-api/scripts/ansible/` install scripts already exist on disk from a prior session but have not been run end-to-end against Vault yet. Phase 0 covers running them.
2. `../../planning/PLAYBOOK-INTEGRATION-PLAN.md` — the full feature plan. Phases 1–10 below map to its "Phases 0–6" section.
3. `../../planning/DATA-MODEL.md` — current entity diagram. Phase 2 adds three tables.
4. `../instructions.md` — the workflow conventions (numbering, folders, /loop directive, SelfImprove discipline).
5. `../craft-ingot/` — the canonical example workflow. All file shapes (`ORCH`, `TASK`, `TEST`, `IMPROVISE`, `RESULT`, `WISHLIST`) are demonstrated there. Mirror them.

## Structure

One master orchestration drives everything:

- **`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`** — the single source of truth. Contains the 11-task checklist and the one `/loop` directive that runs the whole workflow end-to-end. `/loop` exits when every checkbox is checked.
- **`Tasks/0001-...` through `Tasks/0011-...`** — one task per phase, atomic implementation.
- **`Test/0001-...` through `Test/0011-...`** — one test per task, validates that phase.
- **`Improvise/`** — written per-failure during a run (can accumulate many).
- **`Results/0001-...-RESULT.md`** — **one file at the very end**, summarizing the whole workflow.
- **`SelfImprove/0001-...-WISHLIST-001.md`** — **one file at the very end**, reflecting on the whole workflow.

## Phase Index

Phases run in order. Each phase's task is one checkbox in the master orchestration; check it off when its test passes.

| # | Phase | What ships |
|---|---|---|
| 0001 | **Verify install + Vault config** | Run the pre-built `ansible-cli install / configure / verify`; confirm 6/6 smoke tests green |
| 0002 | **Feature flag scaffolding** | `feature_flags` table, `/feature-flags` endpoint, `[RequireFeatureFlag]` attribute, web gating |
| 0003 | **Playbook data model** | `vm.playbooks`, `vm.vm_playbook_assignments`, `vm.playbook_runs` tables + `pb`/`asgn`/`run` prefix registry entries |
| 0004 | **Playbook registry API** | `POST/GET/PATCH/DELETE /api/v1/playbook` — git clone + read `meta/argument_specs.yml` + cache JSON Schema |
| 0005 | **Parameter form (web)** | `@rjsf/core` renders `argument_schema` into a form; saves `vars_override` against assignments |
| 0006 | **Python worker + RabbitMQ + first end-to-end run** | New `cloud-manager-worker` service consumes `playbook-runs` queue, invokes `ansible_runner.run`, no Vault writes yet |
| 0007 | **Vault scoped tokens + outputs** | Worker mints child token via `playbook-run` role, playbook writes to `secrets_prefix`, results surface in UI |
| 0008 | **SignalR run log streaming** | `RunLog_<run_id>` channel streams live stdout to the web UI |
| 0009 | **Schema diffing** | Refresh + apply-time validation against current `git_ref` schema; warn on drift |
| 0010 | **End-to-end smoke: postgres** | Run a real postgres playbook against a fresh VM, verify `admin_password` appears in UI |
| 0011 | **Flip the flag** | `PATCH /api/v1/admin/feature-flags/playbooks {enabled: true}` and confirm UI lights up |

## How To Run

One command drives the whole workflow:

```
Read /home/todd/workspace/cloud-manager/planning/execution-workflow/ansible-shenanigans/init.md and everything it references, then /loop Continue executing tasks in Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md. For each unchecked task in order: read the task file, do the work, run the matching Test/<n> file, write an Improvise on failure and retry until pass, then edit the orchestration in place to check the box. When every box is checked, write Results/0001-ANSIBLE-SHENANIGANS-RESULT.md and SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md and stop.
```

There is no per-phase intervention — `/loop` runs continuously through all 11 phases. If you want to pause or hand off, interrupt and the workflow resumes from the next unchecked box on re-invocation.

## Discipline

- **No phase-skipping silently.** If you must do work out of order (e.g. fix a Phase 2 bug discovered in Phase 6), write an `Improvise/` file documenting why — and don't walk back and rewrite Phase 2's task/test files. The workflow record is a timeline.
- **Reference, don't re-quote.** Task files link to the plan sections; they don't copy them. The plans are the source of truth for *what*; task files own the *how I will execute this in this environment*.
- **Result and Wishlist are workflow-level, not phase-level.** Don't write per-phase result/wishlist files. One Result and one Wishlist, both at the very end, covering all 11 phases.

## Cross-Phase References

- The MCP tools at `mcp__cloud-manager-mcp__*` (vm_list, vm_get, snapshot_*, host_list, vm_images_list, etc.) are the fastest way to inspect API state. Use them when verifying — don't grep DB directly unless the MCP doesn't expose what you need.
- Vault CLI is available; `VAULT_ADDR` and `VAULT_TOKEN` are in the env (see `~/ansible-token.txt` for the install-time token if not set).
- The cloud-manager-api install script is `cloud-manager-api/scripts/api/install-api-service.py` — re-run after any C# change. Idempotent.
