# Result: cloud-manager/ansible-p6-porch

**Outcome: SUCCESS** — all 7 tasks complete, all tests passed, migration applied to live
`clouddb`, API + MCP + porch deployed, deployed-stack smoke green.

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the new dist
> (3 new tools: `cloud_playbook_lint`, `cloud_playbook_check`, `cloud_run_log_chunks`).

## Branch

`ebony-porpoise` (created by hv_next at execution start, 2026-06-12)

## PR table

| Repo | PR | Status |
|------|----|--------|
| cloud-manager-api | TBD (filled after hv_ship) | |
| vorch-lib | TBD | |
| vorch-service | TBD | |
| cloud-manager-mcp | TBD | |
| workflow-exec | TBD | |
| workflow-plans | TBD | |

## Phase summary

| Phase | Task | Result |
|-------|------|--------|
| 0 | Workspace init | `hv_next` → `ebony-porpoise` across 18 repos; branch recorded in deck.md |
| 1 | API: mode + plint/logc | `PlaybookRun.Mode` (varchar(16), default `apply`), `PlaybookLintResult` (`plint`, `vm.playbook_lint_results`, jsonb findings, unique per run), `PlaybookRunLogChunk` (`logc`, `vm.playbook_run_log_chunks`, unique (run,target,seq)); `POST playbook/{id}/lint` → 202+runId publishing `{"runId"}` to NEW `playbook-lints` queue; worker result POST marks run terminal; chunk ingest (≤32KB enforced, 400 over; dup seq → 409) + `?afterSeq=&max=&targetId=` read; run trigger accepts `mode: apply\|check` (400 on bogus); 22/22 scratch-DB behavioral checks + flag-gating checks passed; EF migration `20260612064547_AddPorchExecutionUpgrades` |
| 2 | Porch check mode | `runDTO.Mode` (absent ⇒ apply), `buildRunnerArgs` appends `--cmdline=--check` (single token — FIX-002); `playbook_check_completed` VmEvent per target VM at run roll-up (check runs only); e2e vs pg-test: check run Succeeded + canary NOT created + event recorded; apply run Succeeded + canary created + no event + target-row shape identical to pre-change |
| 3 | Porch lint consumer | vorch-lib: NEW `PlaybookLintMessage` + `PlaybookLintsQueue` const (one new file, zero changed lines elsewhere); porch consumer mirrors collection-install (manual ack, terminal-status ack-and-drop on redelivery — unit-tested with mocked API); `ansible-lint --format json project/` over the materialized tree; violations exit ≠ failure; findings POST → plint row + terminal run + `playbook_lint_completed` global event; e2e: dirty playbook → 5 findings (rule ids match direct ansible-lint output), clean playbook → Succeeded with 0 findings; redaction: 0 `hvs.` hits in findings/logs |
| 4 | Porch chunk streaming | `chunkFlusher` per target: flush at 32KB or 2s, terminal flush, Seq from 0 monotonic per target, POST failures logged-and-tolerated (Seq gap marks loss); tail PATCH untouched; e2e (171KB run output): 7 chunks seq 0..6 reassemble byte-identical to the porch log file, every chunk ≤32KB, tail PATCH = exact last-16KB suffix with pre-change field set, mid-run `afterSeq` paging correct |
| 5 | Health, MCP, regression | Probe additionally runs `ansible-lint --version` → PATCHes `ansible-studio` health (playbooks probe payload untouched — unit-tested both-healthy and lint-missing-degrades-studio-only); 3 MCP tools added to `ansible.ts` per contracts §4, dist rebuilt; parity gate: apply-run PATCH fields/truncation identical to pre-change fixture, `PlaybookRunMessage` reads unchanged; cloud-manager-web zero changes |
| 6 | Migrate + deploy + smoke | below |

## Deploy targets + smoke (deployed stack, HTTP codes)

| Step | Evidence |
|------|----------|
| Migration (live clouddb) | `dotnet ef database update` → applied `20260612064547_AddPorchExecutionUpgrades`; `vm.playbook_runs.mode` default `'apply'`, `plint`/`logc` tables present, `vm_events.virtual_machine_id` nullable |
| API deploy | `deploy-cloud-manager.py --target api` → publish + swap + restart; **up after 4s (HTTP 200)** |
| MCP deploy | `deploy-cloud-manager.py --target mcp` → npm ci + tsc OK; 3 new tools in dist (operator /mcp restart pending) |
| porch deploy | build via fixed `build-app.sh` (FIX-003), local install (FIX-004); `systemctl is-active vorch_service porch` → both active; journal: lint consumer attached to `playbook-lints`, no errors |
| Health | `cloud_health_check` → Healthy; probe-at-boot PATCHed `playbooks` healthy=true AND `ansible-studio` healthy=true with fresh `lastProbedAt` (06:59:23/24Z) |
| Smoke: check run | trigger **202** → `run_7GGHH1CZWE` Succeeded (mode check); canary file ABSENT on pg-test; 1 `playbook_check_completed` event |
| Smoke: lint run | trigger **202** → Succeeded; `plint_5ZBJQYAQ30` with 5 findings (command-instead-of-shell, fqcn[action-core], name[missing], name[play], no-changed-when); 1 `playbook_lint_completed` event |
| Smoke: chunk read | GET **200** → 1 chunk seq [0], ordered (check-run output < 32KB ⇒ single terminal flush) |
| Smoke: apply run | trigger **202** → Succeeded (mode apply); canary created (then cleaned); tail output 6042B with PLAY RECAP — familiar pre-change shape |

## Follow-up notes

1. **ansible-lint provisioning** (Task 0005): `ansible-lint 26.4.0` was installed on the porch
   host via `pipx install ansible-lint` (`PIPX_HOME=/opt/pipx`, `PIPX_BIN_DIR=/usr/local/bin`),
   mirroring the ansible-runner convention. Add this to host provisioning (eng-dev-tools /
   os-image) as a follow-up — intentionally NOT in this plan's diffs (PLAN.md open question 1
   default; the `ansible-studio` health probe is the guard). Optional: add an
   `ANSIBLE_LINT_BIN` Environment line to `install-porch-service.py`'s unit template —
   currently the default `ansible-lint` resolves via systemd's PATH (/usr/local/bin).
2. **vorch-lib version tag** (Task 0003): vorch-service consumes vorch-lib via the go.mod
   `replace` directive; bump the vorch-lib version tag (currently v2.1.60 in go.mod) when the
   vorch-lib PR merges.
3. **ansible-studio flag state**: enabled temporarily for smoke, then **restored to
   enabled=false** (its P1 ship state). Lint/chunk surfaces are 404 until an operator enables
   it; porch's chunk POSTs during apply runs are tolerated failures by design while disabled.
4. **Smoke artifacts**: playbooks `p6-smoke-canary` (pb_3F2DQXY39V) and `p6-smoke-dirty`
   (pb_WSACXZ54ZV) and their runs remain in live clouddb — run/lint FK Restrict prevents
   deletion. Harmless; clean up manually if desired.
5. **vm_events.virtual_machine_id is now nullable**: `playbook_lint_completed` has no VM
   context, so it is recorded as a global (null-VM) event via the new
   `IVmEventService.RecordGlobalEventAsync`. VM timelines filter by VM id and are unaffected.

## FIX files

- FIX-001 — design-time `dotnet ef` targets live clouddb (hardcoded factory); accidental live
  apply during Task 0001 testing, rolled back loss-free; all test applies now use
  `--connection`.
- FIX-002 — `--cmdline --check` as two argv tokens breaks ansible-runner argparse; switched to
  single-token `--cmdline=--check`.
- FIX-003 — `build-app.sh` bit-rotted vs the cmd/ layout (built the module root); now builds
  `cmd/vorch-service` + `cmd/porch`.
- FIX-004 — deploy script's scp/ssh-to-self denied; copy leg executed locally via
  `sudo install` (this host IS the server).
