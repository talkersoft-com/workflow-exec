# CHANGE-0006 — porch install scripts + systemd unit + deploy

## Scripts shipped
Four scripts under `cloud-manager-api/scripts/porch/` matching the vorch/api script style:
- `install-porch-service.py` — `go build -o porch ./cmd/porch`; copies to `/kvm-automator/porch`; writes systemd unit; enables but does NOT start (Phase 7 cutover starts).
- `start-porch-service.py`, `stop-porch-service.py`, `uninstall-porch-service.py` — thin wrappers around systemctl.

## Systemd unit
`/etc/systemd/system/porch.service` created with:
- `Description=Porch — Ansible Playbook Orchestrator`
- `ExecStart=/kvm-automator/porch --config /kvm-automator/config-server.yaml`
- `Environment=RUNNING_AS_SERVICE=true` + `CLOUD_MANAGER_API_URL=http://localhost:5250` + `PLAYBOOK_WORKDIR_ROOT=/kvm-automator/playbook-runs` + `PLAYBOOK_LOG_DIR=/kvm-automator/playbook-logs` + `HOME=/tmp`
- `ReadWritePaths=/kvm-automator /var/lib/cloud-manager /mnt/cloud-storage /usr/share/ansible/collections`
- `Restart=always`, `RestartSec=5s`, `StandardOutput=journal`, `StandardError=journal`
- `User=root` (libvirt + ansible permissions; matches vorch)

## Deviation from PLAN — no separate `/etc/cloud-manager/porch.env`
PLAN Phase 0006 called for a separate env file at `/etc/cloud-manager/porch.env`. The existing project pattern (vorch_service.service, install-vorch-service.py) uses inline `Environment=` lines in the systemd unit + a `config-server.yaml` for Vault/AMQP/DB config. I followed the existing pattern instead of introducing a new env-file convention just for porch. Vault token, AMQP creds, and DB creds all come from `/kvm-automator/config-server.yaml` (already in place from the vorch install).

Net effect: same vars set, same source, fewer moving parts. Easier to grep, easier to roll back.

## Deploy result
- `/kvm-automator/porch` — ELF 64-bit, executable, deployed.
- `porch.service` — registered, enabled, **inactive** (waiting for Phase 7 cutover to start).
- `systemctl is-active porch` → `inactive` (exit 3, the expected systemd code).
