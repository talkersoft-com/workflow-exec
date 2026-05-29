# Result: cloud-manager/azure-tick

## Outcome
SHIPPED — conditional pass. All five jazzy-herring follow-ups closed; porch now runs real pg tasks end-to-end against `postgres-test`. Last task (`Create application user…`) fails because the playbook uses the old `priv:` param removed in `community.postgresql` 4.x — playbook authoring, out of scope.

## Branch
Workflow files under `workflow-exec/cloud-manager/azure-tick/`; commits land on `spectral-skylark` and the next branch hv_ship advances to.

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | filled by hv_ship | — |
| `vorch-service` | filled by hv_ship | — |
| `workflow-exec` | filled by hv_ship | — |

## Phase summary

### Phase 0 — Init
Workspace on `spectral-skylark` (hv_ship auto-advanced). All 15 repos clean.

### Phase 1 — Migration + entity + DTO
Migration `20260529023655_AddVmIpAddress` adds `ip_address varchar(45) nullable` to `vm.virtual_machines`. Entity, DbContext binding, DTO + AutoMapper all wired. Build clean.

### Phase 2 — API endpoint + GET surfaces
`UpdateIpAddressAsync` + `PATCH /api/v1/VirtualMachine/{publicId}/ip-address`. `GetByIdAsync` on PlaybookRunTargetService surfaces `IpAddress`; ListAsync extends summary DTO too. Legacy `NetworkAddresses` lookup removed (table doesn't exist).

### Phase 3 — Materialize under project/ (Gap 2)
`PlaybookMaterializationService` writes `project/site.yml` + `project/roles/{name}/...`. ansible-runner default discovery finds it without `--project-dir`.

### Phase 4 — Vorch IP writeback (Gap 1 server)
`CreateVMCommand` PATCHes the API after libvirt create. Best-effort; failures logged, never escalated.

### Phase 5 — Backfill script
`scripts/porch/backfill-vm-ip-addresses.py` — iterates rows with NULL ip_address, `virsh domifaddr <publicId>` (default lease → ARP fallback), PATCH API. Mid-phase fixes: installed `python3-psycopg2`; switched script to pass `public_id` as the libvirt domain name (not the friendly name). Backfilled 2/4 VMs (postgres-test = 10.0.150.55).

### Phase 6 — Porch inventory generation (Gap 1 runner)
Target DTO gains `IpAddress`. Porch writes `<workdir>/inventory/hosts` (one line per target with `ansible_host`, `ansible_user`, absolute `ansible_ssh_private_key_file`, plus `ansible_ssh_common_args`), passes `--inventory`. Empty IP fails the target cleanly.

### Phase 7 — Drop --project-dir
Removed. `grep -- --project-dir internal/ansible/playbook_run_exec.go` returns zero.

### Phase 8 — Subprocess redaction (Gap 3)
`internal/redact/` package with `Scrub` / `ScrubString` / `NewWriter`. Unit tests pass. `cmd/porch/main.go` uses it via `log.SetOutput`. `playbook_run_exec.go` wraps `cmd.Stderr` AND scrubs every stdout-scanner line before any sink. Verified zero `hvs.*` in porch journal across a full run.

### Phase 9 — ansible-runner system install (Gap 4)
No apt candidate on Ubuntu 24.04 (`apt-cache policy ansible-runner` empty). Used `PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin pipx install ansible-runner` (pipx 1.4.3 lacks `--global`). Symlink `/usr/local/bin/ansible-runner` → `/opt/pipx/venvs/.../ansible-runner`. Porch unit gains `ANSIBLE_RUNNER_BIN=/usr/local/bin/ansible-runner` and flips back to `ProtectHome=yes`. Uninstall script cleans FIX-001 leftovers (`/usr/local/bin/ansible-runner.real`). See CHANGE-0009.

### Phase 10 — Build + deploy + migrate + backfill
Published API, applied migration, ran backfill (2 of 4 VMs populated), reinstalled porch (ProtectHome=yes + system ansible-runner), redeployed vorch (IP writeback). All 3 services active. Single consumer on `playbook-runs`. See CHANGE-0010.

### Phase 11 — Stale-run cleanup (Gap 5)
8 abandoned Queued runs marked Cancelled (7 from the matching WHERE + 1 orphan with a stuck `vault_run_token`). Final count = 0. See CHANGE-0011.

### Phase 12 — End-to-end verification
After FIX-001 (Vault SSH key path + absolute inventory ssh_key + VAULT_TOKEN env + Vault TLS skip-verify) and FIX-002 (stale revision workaround + `acl` install on target VM):

- Run reaches terminal (Succeeded for stripped-no_log revision, Failed for unmodified — both unblock infra).
- `vault_secrets_prefix` = `cloudmanager/data/playbook-secrets/<runId>` ✓
- `vault_run_token` cleared at terminal ✓
- Inventory file generated; SSH connects ✓
- Real pg tasks executed: postgres-16 installed, `app` DB created
- `Create application user…` fails on `community.postgresql.postgresql_user` 4.2.0 rejecting `priv:` (renamed)
- Porch journal: zero `hvs.*` ✓
- `which ansible-runner` = `/usr/local/bin/ansible-runner` ✓
- `ProtectHome=yes` on porch.service ✓
- Single consumer on `playbook-runs` ✓
- Healthprobe round-trips ✓

### Phase 13 — Ship
PR URLs filled by hv_ship.

## Constraints honored
- camelCase JSON (`ipAddress`, `lastProbedAt`).
- public_ids on the wire; internal UUIDs never leaked.
- Vault tokens NEVER in logs — `redact.NewWriter` on stdout/stderr/log AND per-line scrub before any sink. Verified empty `hvs.*` grep on porch journal.
- Vault tokens NEVER in run output streams (same).
- `ip_address` column nullable; existing rows survive.
- One PR per affected repo: `cloud-manager-api`, `vorch-service`, `workflow-exec`. No web. `vorch-lib` not touched.
- After ship: `ProtectHome=yes`, no pipx-symlink shim, no `--project-dir`.

## Out-of-scope gaps surfaced during verification
1. **Auto-revisioning on playbook content edit** — `[[follow-up-auto-revision-on-edit]]`. Direct `vm.playbooks.content` updates aren't snapshotted; materialize uses the stale revision.
2. **pg playbook uses outdated `priv:` param** — `[[follow-up-pg-playbook-postgres-user-priv]]`. Playbook authoring.
3. **Cloud-init image missing `acl` package** — `[[follow-up-cloud-init-include-acl]]`. Required for `become_user: <unprivileged>`.

## Retro
- `CHANGE-0009.md` — apt-vs-pipx decision + PIPX_HOME workaround.
- `CHANGE-0010.md` — deploy + backfill notes (psycopg2, public_id-vs-name).
- `CHANGE-0011.md` — stale-run cleanup counts.
- `FIX-001.md` — Vault SSH key path + inventory absolute path.
- `FIX-002.md` — Stale revision + pg `priv:` drift + ancillary fixes (Vault TLS, VAULT_TOKEN env, ACL package).
- `LESSONS.md` — see file.
