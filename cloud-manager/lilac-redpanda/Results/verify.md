# Browser verification — lilac-redpanda

All six acceptance flows verified against the deployed UI at `https://ubuntu-server.talkersoft.com/ansible/collections`.

## Flow A — Add catalog row ✅
- Clicked `+ Add collection`, filled namespace=`community`, name=`general`, description=`General modules`, submitted.
- New row appeared in the catalog with status "Not installed" and Install button.
- DB: `SELECT public_id, namespace, name FROM ansible.collections WHERE name='general'` → `acoll_KJ674JKWWW | community | general` (single row).
- Screenshot: `Results/flow-a-catalog-add.png`.

## Flow B — Install community.postgresql → Installed ✅
- Clicked Install on the `community.postgresql` row.
- Polling reflected Pending → Installing → Installed within ~30s.
- DB: `install_status=2 (Installed)`, `version=4.2.0`, `install_path=/usr/share/ansible/collections/ansible_collections/community/postgresql`.
- Filesystem: `sudo ls /usr/share/ansible/collections/ansible_collections/community/postgresql` → tree exists.
- Screenshot: `Results/flow-b-installed.png`.

## Flow C — Remove → Removed ✅
- Clicked Remove on community.postgresql.
- Status flipped Removing → Removed within ~5s.
- DB: `install_status=5 (Removed)`, `version=NULL`.
- Filesystem: postgresql directory gone; only `hashi_vault` remained under `/usr/share/ansible/collections/ansible_collections/community/`.
- Screenshot: `Results/flow-c-removed.png`.

## Flow D — Install after Removed → Installed ✅
- Clicked Install on the (now Removed) community.postgresql row.
- Status went Pending → Installing → Installed.
- DB: `install_status=2 (Installed)`, `version=4.2.0`.
- Screenshot: `Results/flow-d-reinstalled.png`.

## Flow E — Install community.hashi_vault → Installed ✅
- Clicked Install on the second seeded row.
- DB after install: `install_status=2, version=7.1.0, install_path=/usr/share/ansible/collections/ansible_collections/community/hashi_vault`.
- Filesystem: both `community/postgresql` and `community/hashi_vault` present under `/usr/share/ansible/collections/ansible_collections/community/`.

## Flow F — pg playbook modules resolve ✅
The original blocker for the `pg` playbook against `postgres-test` was "module not found: community.postgresql.postgresql_db" (and the same for `community.hashi_vault.vault_kv2_write`). After Flow B + E:

```
$ sudo ansible-doc -t module community.postgresql.postgresql_db
> COMMUNITY.POSTGRESQL.POSTGRESQL_DB
  (/usr/share/ansible/collections/ansible_collections/community/postgresql/plugins/modules/postgresql_db.py)
  Add or remove PostgreSQL databases from a remote host.

$ sudo ansible-doc -t module community.hashi_vault.vault_kv2_write
> COMMUNITY.HASHI_VAULT.VAULT_KV2_WRITE
  (/usr/share/ansible/collections/ansible_collections/community/hashi_vault/plugins/modules/vault_kv2_write.py)
  Perform a write operation against a KVv2 secret in HashiCorp Vault
```

Both modules now resolve from the controller-managed collections path. The collection-install feature has cleared this blocker.

(Note: the hashi_vault collection emits a version warning against ansible-core 2.16; that's an upstream compatibility note, not a result of this feature. Acceptance criterion is met — the module is findable; the `pg` playbook will not fail at the module-not-found step.)

Other pg-run blockers identified during this workflow but explicitly NOT in scope for lilac-redpanda:
- Vault scoped-token mint on the cloud-manager-api side (separate plan)
- SSH connectivity to `postgres-test` for the actual playbook task execution

## Sub-nav sanity ✅
Throughout Flows A–D, the Ansible sub-nav showed 5 tabs (Playbooks / Roles / Run / History / Collections) with **Collections** highlighted as active.

## Source-edit scope ✅
End-of-Phase-9 dirty repos are exactly the five named in `deck.md`:
- `cloud-manager-api` (migration + entities + DTOs + services + endpoints + publishers)
- `vorch-lib` (2 message structs)
- `vorch-service` (2 subscribers + 2 handlers)
- `cloud-manager-web` (sub-nav + 2 pages + 3 slices + modal)
- `workflow-exec` (workflow artifacts + 4 FIX files)

No other repos were touched.

## Fix files captured during execution
- `FIX-001` — pre-create `/usr/share/ansible/collections` before vorch start (systemd `ReadWritePaths` requires path existence)
- `FIX-002` — install system-wide `ansible` so root finds `ansible-galaxy` at `/usr/bin/`
- `FIX-003` — set `HOME=/tmp` in the vorch systemd unit so ansible-galaxy can write its tmp dir under `ProtectHome=yes`
- `FIX-004` — always pass `--force` to install at the configured `-p` path even when apt-bundled copies exist; fix the doubled-segment install_path parsing; verify-removal by checking the filesystem path, not `ansible-galaxy collection list`
