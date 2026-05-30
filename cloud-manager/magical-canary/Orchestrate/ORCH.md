# Orchestration: cloud-manager/magical-canary

## Objective

Replace the `/ansible/history` EmptyState with a paginated 50/page grid of every `vm.playbook_runs` row, ordered StartedAt DESC, filterable by VM. The grid is backed by a new `GET /api/v1/Run` list endpoint, surfaced via a new `cloud_playbook_run_list` MCP tool, and bound to URL-driven filter state in the web client. Each row links to `/ansible/runs/:runId` — the existing live/historical run detail view, unchanged. List responses scrub `VaultRunToken`; only `GetByIdAsync` may surface the real token (existing rule, preserved verbatim).

## Inputs

Read these before Task 0000:

- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/gracious-churro/PLAN.md`
- `../../../workflow-plans/cloud-manager/gracious-churro/deck.md`

## Task list

Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; confirm `magical-canary` across all repos
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: API — add `PagedResult<T>` DTO + `PlaybookRunService.ListAsync` with vmId / playbookId / status filters, paging, VaultRunToken scrub
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: API — add `[HttpGet] List` action on `RunController` exposing `GET /api/v1/Run`; preserve feature-flag gate; build + restart api
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: MCP — add `cloud_playbook_run_list` thin pass-through tool; rebuild mcp dist
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: Web — replace `AnsibleHistoryPage` EmptyState with paginated grid + VM combobox; URL ↔ Redux mirror; rows link to `/ansible/runs/:runId`
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: E2E verify — curl API, MCP tool, browser load `/ansible/history`, filter by VM, page, click a row
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: deploy api + mcp + web; write `Results/RESULT.md` + `Retro/LESSONS.md`; `hv_ship`

## Execution steps

1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution

**Option A — MCP (preferred):** Call the following tool:
```
hv_workflow_run  deck: "cloud-manager"  branch: "magical-canary"
```

**Option B — manual:** Type the following command (do not copy-paste):

    workflow execute

DO NOT COPY PASTE — type this command manually. The `workflow` keyword only works when typed directly into the prompt.

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
