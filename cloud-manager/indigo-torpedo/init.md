# Cloud-Manager Ansible Management — planning bundle

## What this workflow does
Writes a five-file planning bundle at
`cloud-manager/planning/planning/cloud-manager-ansible/` that designs full
ansible-management capability across the cloud-manager stack — API, web UI, and
the Go event processor — built on top of the playbook/role tables that already
exist in `clouddb`. The bundle covers cloud-manager-api, cloud-manager-web,
vorch-service (with vorch-lib shared types), and any database deltas needed to
support the new editing surfaces. No code changes — this workflow only produces
markdown design docs.

## Shipping pattern — scaffold ships before execution
The scaffold and the bundle ship on **separate branches**, never co-mingled.
Scaffold review happens against the scaffold PR(s) alone; bundle review happens
against the bundle PR alone.

Lifecycle:
1. **Scaffold PRs** — already merged (execution-workflow#18 on `indigo-torpedo`,
   execution-workflow#19 on `tidy-sandpiper`, execution-workflow#20 on
   `otherworldly-binturong`, and any further scaffold revisions). Each scaffold
   change is its own PR.
2. **Bundle execution ship** — runs on whatever branch `hv_next` produced
   before `/loop` started. Task 0006 calls `hv_ship` once; that single ship
   produces TWO PRs simultaneously, because two repos are dirty by then:
   - `planning` PR — the 5-file bundle under `cloud-manager-ansible/`.
   - `execution-workflow` PR — `Results/RESULT.md` + `Retro/LESSONS.md` written
     into this workflow folder per the project's workflow convention.
   Both are expected, both are part of the bundle ship.

The workflow folder name `indigo-torpedo` is the **workflow ID**, NOT the
branch the bundle ships on. The bundle's branch is whatever `hv_next` produced
before `/loop` started; Task 0000 records it in `deck.md` at runtime.

## Read before starting
- `deck.md` — repos in scope (`planning` for the bundle, `execution-workflow` for Results+Lessons); hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../../planning/DATA-MODEL.md` — the deployed entity-relationship model (source of truth)
- `../../../planning/PLAYBOOK-INTEGRATION-PLAN.md` — partially stale (still references git-backed playbook fields and assignment-tied PlaybookRun); the new bundle supersedes the data-model and worker sections
- `../../../planning/ANSIBLE-INSTALL-PLAN.md` — still authoritative for ansible-runner + Vault policy install

## Pre-shipped facts the plans must respect
- `clouddb` (Postgres 17, localhost:5432) has all 16 ansible-related tables in
  the `vm` schema plus `membership.feature_flags`. Six EF migrations applied
  through `AddPlaybookManagement`. Verified live via the database-toolkit MCP.
- `Playbook.content` (text) is the source of truth. The `playbooks` table no
  longer has `git_url`, `git_ref`, `playbook_path`, or `last_refreshed_at`.
- `PlaybookRun` owns execution via `playbook_id` + `playbook_revision_id`, not
  `assignment_id`. Per-VM targeting lives on `PlaybookRunTarget`.
- `RunController.Apply` already calls `IPlaybookRunPublisher.EnqueueRun(runId)`.
  Nothing consumes the queue yet — that's the gap `VORCH-PLAN.md` fills.
- `ArgumentSpecsTranslator` and `SchemaDiff` already exist at
  `src/Services/CloudManager.Data.Services/Playbooks/` and are unused.
- `EntityPrefixRegistry` already has prefixes for every new entity.
- `cloud-manager-worker/` is an empty Python scaffold from PLAYBOOK-INTEGRATION-PLAN.md
  Phase 3. The VORCH-PLAN must decide whether to absorb or delete it.

## Constraints
- No plan may reintroduce git-backed playbook fields (`git_url`, `git_ref`,
  `playbook_path`, `last_refreshed_at`).
- Plans must use the existing `[RequireFeatureFlag("playbooks")]` attribute for
  every new playbook/role endpoint.
- Plans must reuse existing public_id prefixes from `EntityPrefixRegistry` (`pb`,
  `pbrev`, `ansr`, `rf`, `rfrev`, `pgr`, `plr`, `plrf`, `plrfr`, `prt`, `asgn`,
  `run`, `ff`) — do not invent new ones unless `DATA-MODEL-DELTAS.md` justifies it.
- Each plan ends with a `## Phases` section ordered from "smallest shippable
  slice" to "complete feature".
- No code is written by this workflow. Implementation happens in follow-on
  workflows scoped to each plan.
