# Execution: cloud-manager/ansible-p6-porch

## Objective
Porch gains three execution-safety capabilities, all additive on the existing RabbitMQ +
runId machinery, with `PlaybookRunMessage` and the materialization structs in vorch-lib
byte-identical after the work: (1) **check mode** — `POST api/v1/playbook/{pid}/run` accepts
optional `"mode": "apply" | "check"` (default `apply`), persisted as `PlaybookRun.Mode`
(additive column, `vm` schema), exposed in run DTOs; porch appends `--cmdline "--check"` to
the ansible-runner invocation when the fetched run says `check`, with Vault, inventory,
redaction, and status writes untouched, plus an additive `playbook_check_completed` event.
(2) **ansible-lint** — `POST api/v1/playbook/{pid}/lint` → 202 + runId via a NEW
`playbook-lints` queue and NEW `PlaybookLintMessage{ runId }` in vorch-lib; a new porch
consumer mirroring the collection-install pattern (manual ack, terminal-status check on
redelivery) materializes the playbook, runs `ansible-lint --format json project/`, POSTs
findings to `POST api/v1/playbook/{pid}/lint/{runId}/result` (`plint` entity, jsonb findings
per playbook revision), and marks the run terminal, plus an additive
`playbook_lint_completed` event; the health probe additionally checks
`ansible-lint --version` and PATCHes `ansible-studio` flag health. (3) **chunked log
streaming** — `PlaybookRunLogChunk` (`logc`, `vm` schema; RunId, TargetId, monotonic Seq per
target, Content ≤32KB) with worker-auth ingest `POST api/v1/run/{runId}/log/chunk` and read
`GET api/v1/run/{runId}/log/chunk?afterSeq=&max=`; porch flushes redacted sequenced chunks
(per 32KB or 2s) BESIDE the existing 16KB-tail PATCH, which is parity-tested byte-identical.
MCP gains `cloud_playbook_lint`, `cloud_playbook_check`, `cloud_run_log_chunks` in
`ansible.ts`. Migration applied, api + mcp + porch built and deployed, shipped on one
exec-stage PR set. No web changes; no change to any existing vorch-lib struct, JSON tag,
queue name, or the five status strings.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p6-porch/PLAN.md` — especially the additive-only audit table
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §1 (`plint`/`logc`), §3, §4 (MCP), §6 (events), §11 (porch additive-only is a series-wide invariant; health probes)
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §6, §10
- vorch-service `internal/ansible/` + vorch-lib `models/` — current pipeline, message structs, collection-install consumer pattern

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: API — run `Mode` column, `plint` + `logc` entities/migrations, lint/chunk endpoints, `playbook-lints` publish, run DTO `mode`
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: porch check mode — `--cmdline "--check"` off the fetched run mode, check event, e2e against a real VM
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: porch lint consumer — vorch-lib `PlaybookLintMessage` + queue, lint handler, findings POST, terminal handling
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: porch chunk streaming — sequenced chunk flush beside the existing tail PATCH; ordering + redaction tests
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: health probe (`ansible-lint` → `ansible-studio`), MCP tools, tail-PATCH parity regression
- [ ] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api + mcp + porch → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p6-porch"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
