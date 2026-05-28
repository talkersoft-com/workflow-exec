# Result: cloud-manager/lilac-redpanda

## Outcome
**SHIPPED** (pending the final hv_ship call below)

## Branch
`exuberant-harp` (execution; planning was on `lilac-redpanda`)

## Pull Requests
Filled in from the hv_ship output at the end of this task.

| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | TBD | — |
| `vorch-lib` | TBD | — |
| `vorch-service` | TBD | — |
| `cloud-manager-web` | TBD | — |
| `workflow-exec` | TBD | — |

## Phase summary

### Phase 0 — Initialize
Deck on `exuberant-harp`, all 15 repos clean. Branch recorded in `deck.md`.

### Phase 1 — Migration
New `ansible` schema. Four tables + `CollectionInstallStatus` enum. Seeded `community.postgresql` + `community.hashi_vault`. `dotnet ef database update` applied cleanly. See `Retro/CHANGE-0001.md`.

### Phase 2 — DTOs + Mapper + Services + DI
Four DTOs, AnsibleCollectionProfile, IAnsibleCollectionService / IHostCollectionService / ICollectionInstallRunService + implementations, DI registration. Build clean. See `Retro/CHANGE-0002.md`.

### Phase 3 — API endpoints
Three controllers, ten endpoints under `/api/v1/ansible`. See `Retro/CHANGE-0003.md`.

### Phase 4 — AMQP publishers
`CollectionInstallPublisher` + `CollectionRemovePublisher`, DI'd, wired into `HostCollectionsController`. Bodies `{ "runId": "acrun_..." }` (camelCase). See `Retro/CHANGE-0004.md`.

### Phase 5 — vorch-lib messages
`CollectionInstallRunMessage` + `CollectionRemoveRunMessage` with camelCase JSON tags. `go build` + `go vet` clean. See `Retro/CHANGE-0005.md`.

### Phase 6 — vorch subscribers + handlers + ReadWritePaths widen
Two subscribers + two handlers; ansible-galaxy invocation; safety-guard prefix check on remove. `install-vorch-service.py` widens `ReadWritePaths` to include `/usr/share/ansible/collections`. See `Retro/CHANGE-0006.md`.

### Phase 7 — Web Collections tab
5th sub-nav tab; AnsibleCollectionsPage (list with state-aware actions + Add modal); AnsibleCollectionDetailPage; three Redux slices including `pollUntilTerminal`. `tsc --noEmit` + `npm run build` clean. See `Retro/CHANGE-0007.md`.

### Phase 8 — Build + deploy
Migration applied. All three services rebuilt and redeployed. FIX-001 (pre-create the collections path so systemd accepts the unit) caught in this phase. All three services `active`. Catalog endpoint returns the two seeded rows.

### Phase 9 — Playwright e2e
Walked all six flows. Screenshots: `post-deploy-list.png`, `flow-a-catalog-add.png`, `flow-b-installed.png`, `flow-c-removed.png`, `flow-d-reinstalled.png`. Full notes in `Results/verify.md`.

Three additional FIX files captured during this phase:
- `FIX-002` — installed system-wide `ansible` so root finds `ansible-galaxy`
- `FIX-003` — set `HOME=/tmp` in the vorch systemd unit (ansible-galaxy needs writable HOME under `ProtectHome=yes`)
- `FIX-004` — always use `--force` on install (to override apt-bundled copies); fix doubled-segment install_path parsing; verify-removal by filesystem path absence rather than `ansible-galaxy collection list` (which still sees apt-bundled copies)

### Phase 10 — Ship
This task. `hv_ship` produces 5 PRs.

## Key acceptance results
- Catalog rows: 3 (2 seeded + community.general added during Flow A)
- Both `community.postgresql` and `community.hashi_vault` end-state Installed at `/usr/share/ansible/collections/ansible_collections/community/{name}`
- `ansible-doc -t module community.postgresql.postgresql_db` resolves to the controller-managed path
- `ansible-doc -t module community.hashi_vault.vault_kv2_write` resolves to the controller-managed path
- The original `pg` module-not-found blocker is cleared

## Out-of-scope (carried forward)
- Vault scoped-token mint on cloud-manager-api side (separate plan)
- `playbook_collection_requirements` enforcement in the Run trigger flow (schema-only this PR set)
- Multi-controller / multi-host
- Galaxy API integration for catalog metadata
