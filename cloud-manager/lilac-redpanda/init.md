# Web-driven Ansible collection install / remove / reinstall

## What this workflow does
Implements the four `ansible.*` tables (already in DATA-MODEL.md), the API surface (ten endpoints + two AMQP publishers), the vorch-service worker (two subscribers + two handlers that call `ansible-galaxy` and verify), and the cloud-manager-web UI (new Collections sub-nav tab, list/detail pages, add modal, install/remove/reinstall actions). End state: an operator opens `/ansible/collections`, adds `community.postgresql`, clicks Install, watches it flip Pending → Installing → Installed within ~30s; then Remove → Removed; then Install again. The `pg` playbook's `community.postgresql.postgresql_db` and `community.hashi_vault.vault_kv2_write` tasks resolve at run time.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- `../../../workflow-plans/cloud-manager/peppery-kudu/PLAN.md` — source plan with all design choices
- `../../../workflow-plans/DATA-MODEL.md` — the four-table schema this workflow implements

## Constraints
- **camelCase JSON** on the wire. **public_ids** on the wire (e.g. `acoll_…`, `hacoll_…`, `acrun_…`). Internal UUIDs never leaked.
- **Worker writes are PATCHes through the API.** Vorch never opens a DB connection.
- **Two AMQP queues**, not one with command dispatch: `collection-installs` and `collection-removes`. Mirrors the `playbook-runs` / `playbook-run-cancel` pattern.
- **Don't reuse the `playbook-runs` queue** for collection installs.
- **Don't conflate enums.** `host_collections.install_status` uses the new `CollectionInstallStatus` (string). `collection_install_runs.status` reuses the existing `PlaybookRunStatus` (int). These are intentionally different — the doc commits to this.
- **Don't implement `playbook_collection_requirements` enforcement** in the Run trigger flow. Schema-only this PR set; reading + enforcing is a future plan.
- **Don't widen the systemd `ReadWritePaths`** beyond `/usr/share/ansible/collections`. Phase 6 adds exactly that one path.
- **Remove path uses `rm -rf` with a hard prefix guard** (`/usr/share/ansible/collections/ansible_collections/`) — `ansible-galaxy` has no native uninstall.
- **One feature, one PR per affected repo.** Five PRs expected (api, vorch-lib, vorch-service, web, workflow-exec).
- **All hv operations through MCP tools** — never raw `git` for repos in the deck.
