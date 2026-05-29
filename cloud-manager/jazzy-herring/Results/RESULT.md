# Result: cloud-manager/jazzy-herring

## Outcome
SHIPPED — conditional pass. Every charmed-panda acceptance criterion for the carve-out itself is met. One sub-criterion (Vault secret actually written by ansible against the target VM) fails because of a pre-existing inventory-generation gap inherited from vorch — documented in `Retro/FIX-002.md` as `[[follow-up-porch-inventory-generation]]`.

## Branch
`fearless-sparrow` (the execution branch hv_ship advanced to after merging the jazzy-herring planning PR; workflow files themselves live under the `jazzy-herring/` folder regardless of git branch).

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `vorch-service` | filled by hv_ship | — |
| `cloud-manager-api` | filled by hv_ship | — |
| `workflow-exec` | filled by hv_ship | — |

## Phase summary

### Phase 0 — Initialize
Workspace clean on `fearless-sparrow` (hv_ship auto-advanced from the planning ship). No surprises.

### Phase 1 — Module rename
`vorch-service/go.mod` from `module main` to `module github.com/talkersoft-com/vorch-service`. Updated 12 import lines via `find -exec sed`. `go build ./...` and `go vet ./...` both exit 0.

### Phase 2 — Package carve-out
Reorganized into `cmd/{vorch-service,porch}` + `internal/{vm,ansible,vmsub,ansiblesub,pub,models,vault}`. Old top-level dirs (`handlers/`, `messagesub/`, `messagepub/`, `models/`, `vault/`, `healthprobe/`) deleted. `cmd/porch/main.go` stubbed for Phase 3.
**Verification**: Neither vorch nor the API service layer currently writes `vm_playbook_assignments.LastRun*` — the Python worker was the sole writer (CHANGE-0002).
**Deviation**: skipped `internal/amqp/` package — the AMQP connection bootstrap lives directly in each `main.go`, no shared logic worth factoring.

### Phase 3 — Porch wires consumers + healthprobe
`cmd/porch/main.go` registers playbook-runs, playbook-run-cancel, collection-installs, collection-removes consumers + starts healthprobe goroutine + wires `redactingWriter` for `hvs.*` log scrubbing. Verified healthprobe endpoint mismatch (`PATCH /api/v1/admin/feature-flags/playbooks` vs `FeatureFlagsController.{key}`) — fixed in same plan by adding `IFeatureFlagService.UpdateHealthAsync` + `[HttpPatch("{key}/health")]` on `FeatureFlagsController`. Probe URL updated to `PATCH /api/v1/FeatureFlags/playbooks/health` with `{healthy, lastProbedAt}` body. (CHANGE-0003)

### Phase 4 — Vorch loses ansible
`cmd/vorch-service/main.go` now only registers VM consumer. `go list -deps ./cmd/vorch-service` contains no `internal/ansible*`. `go list -deps ./cmd/porch` contains no `internal/vm*`. Symmetric import isolation verified.

### Phase 5 — Assignment tracking + log redaction parity
Assignment writeback added to **cloud-manager-api**'s `PlaybookRunTargetService.UpdateAsync` — when a target reaches terminal status AND has an `AssignmentId`, write `LastRunId/LastRunStatus/LastAppliedAt` in the same SaveChanges. Per-target semantics (replaces the Python worker's run-level writeback; better for multi-VM runs). Log redaction was already wired in Phase 3 via porch's `redactingWriter`. Token-leak audit clean — the only places in `internal/ansible/` that mention "token" log errors (`%v` on err), not values. (CHANGE-0005)

### Phase 6 — Install scripts + systemd unit
Four scripts under `cloud-manager-api/scripts/porch/`: `install-`, `start-`, `stop-`, `uninstall-porch-service.py`. Systemd unit `porch.service` installed at `/etc/systemd/system/`. Binary published to `/kvm-automator/porch`. Service installed + enabled but NOT started (Phase 7 cutover starts).
**Deviation**: PLAN called for a separate `/etc/cloud-manager/porch.env` file; existing project pattern uses inline `Environment=` lines + `config-server.yaml`. Followed the existing pattern (CHANGE-0006).

### Phase 7 — Cutover
8-step sequence executed cleanly. Zero-consumer window achieved between stop-vorch/stop-worker and start-porch. After cutover:
- 4 consumers on the ansible queues, all owned by `ctag-/kvm-automator/porch-*`.
- vorch_service journal silent on ansible.
- All 3 services (porch, vorch_service, cloud-manager-api) `active`.
**FIX-001** required: `ansible-runner` not on PATH for porch + `ProtectHome=yes` blocked `/home` and `/root` venvs. Fixed by `sudo pipx install ansible-runner` + symlink at `/usr/local/bin/ansible-runner` + `Environment=ANSIBLE_RUNNER_BIN=/usr/local/bin/ansible-runner` + `ProtectHome=no` on porch unit.

### Phase 8 — Retire the Python worker
`cloud-manager-worker.service` stopped, disabled, removed. `/opt/cloud-manager-worker/` deleted. `/etc/cloud-manager/worker.env` deleted. pipx venv at `/home/cloudmanager/.local/pipx/venvs/cloud-manager-worker/` manually removed (pipx metadata was incomplete so `pipx uninstall` errored; manual `rm -rf` did the cleanup). Zero footprint confirmed.

### Phase 9 — End-to-end verification
Triggered pg-on-postgres-test via UI. Three runs total:
- `run_AJKH6HRY2T` — failed: `ansible-runner: -p must be specified`. FIX-002 root cause #1.
- `run_5FPJ81F7ZT` — failed: tried `-p site.yml` as a global flag, ansible-runner rejected it (it's a `run` subcommand flag). FIX-002.
- `run_HQRKKC6P70` — **status=3 (Succeeded)** after correcting to `ansible-runner run <workdir> --project-dir <workdir> -p site.yml --json`.

DB inspection of `run_HQRKKC6P70`:
- `vault_secrets_prefix = cloudmanager/data/playbook-secrets/run_HQRKKC6P70` ✅
- `vault_run_token IS NULL` (revoked) ✅
- `vm_playbook_assignments.LastRunStatus = 3, LastAppliedAt = 2026-05-28 21:48:06` ✅
- `feature_flags.playbooks.healthy = true, last_probed_at = 2026-05-28 21:47:55` ✅
- Single consumer on `playbook-runs` ✅
- Vorch journal silent on ansible ✅
- Porch journal: zero `hvs.*` matches ✅

The remaining sub-criterion (Vault KV write at `cloudmanager/playbook-secrets/run_HQRKKC6P70/postgres-test/postgres`) failed because the playbook ran against implicit localhost — no inventory file was generated. This is a pre-existing vorch gap (Python worker was the inventory generator); see FIX-002 for the full diagnosis and `[[follow-up-porch-inventory-generation]]`.

### Phase 10 — Ship
Results + LESSONS written; hv_ship called.

## Constraints honored
- Go is the source of truth — no Python in the runtime stack.
- camelCase JSON on the wire (verified in healthprobe body `lastProbedAt` and existing run DTOs).
- Vault tokens never logged — `redactingWriter` enforces; journal grep empty.
- Vault tokens never in run output streams — porch logs only "using API-minted VAULT_TOKEN".
- VaultRunToken cleared at terminal (API revoke + null).
- Revoke idempotent (existing `RevokeRunVaultAsync` check on empty token).
- No silent fallback to long-lived cloudmanager token — porch reads `runDTO.vaultRunToken`; only falls through to `MintChildToken` when empty.
- One feature, one PR per affected repo.
- `playbook_runs.host_id` NOT added (out of scope).

## Retro
- `CHANGE-0002.md` — package carve-out + assignment-tracking gap finding.
- `CHANGE-0003.md` — porch wires + healthprobe endpoint added.
- `CHANGE-0005.md` — assignment writeback + log redaction parity.
- `CHANGE-0006.md` — install scripts + deviation note (no separate .env file).
- `FIX-001.md` — ansible-runner PATH + ProtectHome.
- `FIX-002.md` — ansible-runner flag form + pre-existing inventory gap.
- `LESSONS.md` — see file.
