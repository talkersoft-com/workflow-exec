# Deck: cloud-manager

## Deck name
`cloud-manager`

## Orchestration id (plan codename)
`syrupy-jadeplant`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `vorch-lib` | `provision/provision.go` `buildUserData` — append the lockout-safe `ufw` block (allow `22/tcp` → `--force default deny incoming` / `--force default allow outgoing` → `--force enable`) to the user-data `runcmd`, after `netplan apply`. Golden/unit test in `provision/provision_secrets_test.go`. |
| `workflow-exec` | This orchestration folder (runtime) |

## Non-repo state changed (cloud-manager DB, via MCP)
The Postgres / RabbitMQ playbooks are edited through the **cloud-manager-mcp** `cloud_playbook_*`
tools (DB-authoritative), not as files in a deck repo. Phase 0002 records (open question in PLAN)
whether they are DB-only or seeded from a repo; if a repo turns out to back them, add a
`## Manual steps` section here with explicit git instructions for that repo.
- Postgres: `pb_PG14JAMMY01`, `pb_EMEJCH08GG` → `ufw allow from {{ ufw_sources }} to 5432`.
- RabbitMQ (jammy/noble) → `5672` (+ `15672` only if mgmt UI wanted on-subnet).

**Must NOT change:** `vorch-service` (no new create-vm field — baseline is static), `cloud-manager-api`,
`cloud-manager-web`, `cloud-manager-cli`, the DB schema (no migration), and every other repo in the
deck.

## Branch
`springy-snowflake` — execution branch minted by the warm-up `hv_next` on 2026-06-26 (from
origin/hive, 16 repos). Task 0000 re-verifies and records it here; no second branch is created.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"   # only if clean on a feature branch with all PRs merged
```

## Vorch build + manual redeploy (Phase 0003, BEFORE integrate)
The cloud-manager `.cicd` build/deploy scripts target api/mcp/web only — they do NOT build vorch.
Build vorch and redeploy it **manually on the hypervisor** so the running provisioner emits the new
user-data:
```
cd vm-infra/cloud-manager/vorch-lib    && go build ./... && go test ./provision/...
cd vm-infra/cloud-manager/vorch-service && ./scripts/build-app.sh && ./scripts/deploy-app.sh
```
> Note: `deploy-app.sh` cannot ssh-to-self — install/run the built binary locally on the hypervisor
> (see project memory `dotnet-ef-live-db` / vorch deploy notes). Record exact commands run in
> `Results/RESULT.md`.

## Integrate / ship (final phase, AFTER deploy, BEFORE done)
```
hv_integrate  deck:    "cloud-manager"
              message: "feat: default-deny ufw host firewall baseline + per-service subnet allow (syrupy-jadeplant)"
              title:   "Default-deny host firewall (ufw) for provisioned VMs (syrupy-jadeplant)"
              body:    "Security #5. vorch-lib cloud-init ufw baseline (SSH-only by default); Postgres/RabbitMQ playbooks open their own port to the trusted subnet. No api/web/db changes; no migration."
```
PRs are queued for auto-merge; `hv_integrate` detects the merges and transitions the deck
automatically.
