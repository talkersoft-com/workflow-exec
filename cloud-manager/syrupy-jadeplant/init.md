# Default-deny host firewall (ufw) for provisioned VMs (syrupy-jadeplant)

## What this workflow does
Workflow #5 of the security-hardening track. Today every provisioned VM boots with **no host
firewall**, so any listening service is exposed to anything that can route to it (proven: an
off-subnet Mac reached Postgres `5432` directly — only `pg_hba` stopped the auth). This workflow
makes a fresh VM **SSH-only by default** and lets each service playbook open its own port to the
trusted VM subnet.
- **vorch-lib (cloud-init baseline)** — append a lockout-safe `ufw` block to the user-data `runcmd`
  (after netplan apply): allow `22/tcp` → default deny incoming / allow outgoing → enable. Applies
  to every **newly provisioned** VM. No new create-vm field (static template → vorch-service
  unchanged).
- **Service playbooks (per-port subnet allow)** — Postgres (`pb_PG14JAMMY01`, `pb_EMEJCH08GG`,
  port `5432`) and RabbitMQ (jammy/noble, `5672` + `15672` mgmt only if wanted) add a `ufw allow
  from {{ ufw_sources }} to <port>` task, mirroring the existing `pg_hba_sources`.

## Read before starting
- `deck.md` — scope, repos that change, and hv MCP calls
- `Execution/Exec.md` — task list
- `../../../workflow-plans/cloud-manager/syrupy-jadeplant/PLAN.md`

## Constraints
- **No api/web/db changes; no migration.** The baseline is static in the user-data template, so
  **vorch-service is unchanged** and there is no new create-vm field.
- **Lockout-safe order is mandatory:** allow `22/tcp` BEFORE `ufw enable`. Established connections
  are preserved and 22 is open, so post-boot porch/ansible SSH runs are unaffected.
- **Subnet stays reachable on-subnet:** keep Postgres reachable from `10.0.150.0/24` +
  `10.0.144.140/32` (peer VMs need it) — do not revert to localhost-only. Off-subnet/operator
  access goes via SSH tunnel (the separate #6 work).
- **No secret/value logging** — secret-injection logic is unchanged; the empty-secrets user-data
  path stays byte-stable apart from the new ufw block.
- Build vorch and **manually redeploy on the hypervisor**; verify E2E against a real provision.
