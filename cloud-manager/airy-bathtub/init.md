# Orchestration: cloud-manager/airy-bathtub

## Source plan
`../../../workflow-plans/cloud-manager/airy-bathtub/PLAN.md`
(Default-deny host firewall (ufw) for provisioned VMs — v2, decision-complete.
Supersedes the `syrupy-jadeplant` firewall plan.)

## What this orchestration delivers
Lock provisioned VMs down at the network layer. Today every VM boots with **no host
firewall**, so any listening service is reachable by anything that can route to it. After
this orchestration:

- **Every** newly provisioned VM boots SSH-only: port `22/tcp` allowed from anywhere,
  default **deny incoming / allow outgoing**, ufw enabled — installed by the vorch
  cloud-init user-data builder.
- **Each service playbook opens its own port to the trusted subnet only:**
  - Postgres seed playbooks (`pg-14-jammy.yaml`, `pg-16-noble.yaml`) gain a ufw task
    opening `5432` from the trusted sources.
  - RabbitMQ seed playbooks (`rabbitmq-jammy.yaml`, `rabbitmq-noble.yaml`) tighten their
    existing open-to-anywhere ufw task to the trusted sources for `5672` + `15672`.
- Seed YAML edits are pushed to the live DB via `cloud-manager-mcp/.cicd/import-playbooks.py`.

## Trusted sources
`10.0.150.0/24` + `10.0.144.140/32` (reuse the Postgres playbook's `pg_hba_sources`).

## Repos touched
- `vorch-lib` — cloud-init user-data ufw baseline (Go).
- `cloud-manager-api` — seed playbook YAMLs (`seed/playbooks/*.yaml`).

No api/web/db-schema changes; no migration. **vorch-service unchanged** (rule is static in
the template; no new create-vm field).

## Scope guardrails (from the plan)
- Only newly provisioned VMs get the baseline (cloud-init first boot). Do **not** retro-fire
  existing VMs.
- Do **not** revert Postgres to localhost-only — peer VMs need direct subnet access.
- SSH stays open from `0.0.0.0/0` (cloud image is key-only; operator roams networks). Final.

## Execution
This is a decision-complete plan. The exec agent runs it end-to-end without pausing for
operator input — resolve any ambiguity from the codebase, never defer.

Read `deck.md` next, then `Execution/Exec.md`.
