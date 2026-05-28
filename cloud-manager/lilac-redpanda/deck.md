# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | EF Core migration (`ansible` schema, 4 tables, `CollectionInstallStatus` enum, seed); entities + DTOs + AutoMapper profile; 10 endpoints under `/api/v1/ansible`; `CollectionInstallPublisher` + `CollectionRemovePublisher`; DI registration |
| `vorch-lib` | `CollectionInstallRunMessage` + `CollectionRemoveRunMessage` structs (camelCase JSON tags) |
| `vorch-service` | `messagesub/collection_install_sub.go` + `messagesub/collection_remove_sub.go`; `handlers/collection_install_handler.go` + `handlers/collection_remove_handler.go`; `install-vorch-service.py` widens `ReadWritePaths` to include `/usr/share/ansible/collections` |
| `cloud-manager-web` | `AnsibleSubNav.tsx` adds Collections tab; `AnsibleCollectionsPage` (list) + `AnsibleCollectionDetailPage` (detail) + add modal; three Redux slices (`collectionsSlice`, `hostCollectionsSlice`, `collectionInstallRunsSlice`); polling for transitional states |
| `workflow-exec` | Always — workflow scaffold + Results/Lessons at ship time |

Repos not listed will be on the feature branch but skipped by `hv_ship`.

## Branch
`exuberant-harp` (execution; planning was on `lilac-redpanda`)

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat: web-driven Ansible collection install / remove / reinstall"
         title:   "feat: web-driven Ansible collection install / remove / reinstall"
```

## Operational notes
- Workflow runs ON `ubuntu-server.talkersoft.com`. Live API at `localhost:5250`, web at `localhost:3000` (nginx at `https://ubuntu-server.talkersoft.com`).
- Passwordless `sudo` available for `journalctl`, `systemctl`, `install-*.py`, and EF `dotnet ef database update`.
- Deploy scripts:
  - API: `cloud-manager-api/scripts/api/install-api-service.py`
  - Vorch: `cloud-manager-api/scripts/vorch/install-vorch-service.py`
  - Web: `cloud-manager-api/scripts/web/install-web-app.py` (always `sudo rm -rf cloud-manager-web/dist` first)
- DB access: `database-toolkit` MCP, `clouddb` at localhost:5432.
- `ansible-galaxy` is installed (came with `ansible-core` via `ansible-runner` install earlier in the session).
- Path the worker will write to: `/usr/share/ansible/collections/ansible_collections/<namespace>/<name>` — same path `ansible-runner` reads from by default.
- The `pg` playbook (`pb_EMEJCH08GG`) references the two collections this feature installs; final verification is a `pg` run that no longer fails with "module not found."
