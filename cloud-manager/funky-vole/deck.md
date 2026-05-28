# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | EF migration (`vm.playbook_runs.vault_run_token` + `bare_metal.hosts.is_controller` + seed); new `CloudManager.Vault.Client` project; `PlaybookRunService.TriggerMultiVmAsync` mint+enforce; `RevokeRunVaultAsync` on terminal; `VaultRunTokenSweeper` `BackgroundService`; redaction log enricher; `PlaybookCollectionRequirement` service + 3 endpoints; `Host.IsController` entity + DTO + API surface |
| `vorch-service` | runDTO adds `vaultRunToken`; `playbook_run_exec.executeOneTarget` sets `VAULT_TOKEN` + `VAULT_ADDR` env on the subprocess |
| `cloud-manager-web` | selected-controller persistence; controller dropdown on `AnsibleCollectionsPage` + `AnsibleCollectionDetailPage` + "All controllers" summary; `playbookCollectionRequirementsSlice` + Required-collections panel on `PlaybookDetailPage` + Add-requirement modal; `AnsibleRunPage` blocks Run when requirements unmet |
| `workflow-exec` | workflow scaffold + Results/Lessons at ship time |

Repos not listed will be on the feature branch but skipped by hv_ship. Note: `vorch-lib` is **not** touched in this workflow — `runDTO` lives in `vorch-service`.

## Branch
`fizzy-meerkat` (execution; planning was on `funky-vole`)

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
         message: "feat: Vault scoped tokens + requirement enforcement + multi-controller surface"
         title:   "feat: Vault scoped tokens + requirement enforcement + multi-controller surface"
```

## Operational notes
- Workflow runs ON `ubuntu-server.talkersoft.com`. API at `localhost:5250`, web at `localhost:3000` behind nginx.
- Vault at `https://ubuntu-server.talkersoft.com:8200`; the cloudmanager token in `/etc/cloud-manager/cloud-manager-config.env` has full `cloudmanager/*` access and can mint child tokens via `auth/token/create`.
- Deploy scripts in `cloud-manager-api/scripts/{api,vorch,web}/install-*.py`. Always `sudo rm -rf cloud-manager-web/dist` before web rebuild.
- DB access via `database-toolkit` MCP or `psql -h localhost -U cloudmanager -d clouddb` (password `P@ssw0rd!`).
- `dotnet ef database update -s ../../CloudManager.API` from `Models/CloudManager.Entities` to apply migrations. Env vars: `DBHOST=localhost DBUSER=cloudmanager DBPASSWD='P@ssw0rd!' DBPORT=5432 DATABASE=clouddb RABBITMQ_HOST=localhost RABBITMQ_USER=cloudmanager RABBITMQ_PASSWORD='P@ssw0rd!'`.
- For acceptance: `community.postgresql` + `community.hashi_vault` are already installed on `ubuntu-server` from lilac-redpanda.
