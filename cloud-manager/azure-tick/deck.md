# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | Migration (`ip_address` on `vm.virtual_machines`); entity + DTO + AutoMapper; new `PATCH /api/v1/vm/{publicId}/ip-address`; extend GET DTOs with `ipAddress`; `PlaybookMaterializationService` writes under `project/`; updated `scripts/porch/install-porch-service.py` + `uninstall-porch-service.py` + new `backfill-vm-ip-addresses.py`. |
| `vorch-service` | IP writeback HTTP call from `CreateVMCommand`; inventory generation in `executeOneTarget` (`<workdir>/inventory/hosts`); `redactingWriter` moved to `internal/redact/` + applied to subprocess streams + log publisher; `--project-dir` flag dropped. |
| `workflow-exec` | This deck (azure-tick). |

Repos not listed (`vorch-lib`, `cloud-manager-web`, `cloud-manager-mcp`, etc.) will be on the feature branch but skipped by hv_ship.

## Branch
`azure-tick`

## Initialize (Task 0000)
Workspace was already advanced to `azure-tick` by the planning ship of nutty-cobbler. Confirm:
```
hv_status  deck: "cloud-manager"
```
If somehow not on `azure-tick`:
```
hv_next  deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "porch: inventory generation, redactor for subprocess streams, ansible-runner system install, materialize under project/"
         title:   "porch: pg playbook E2E — close all jazzy-herring follow-ups"
```
