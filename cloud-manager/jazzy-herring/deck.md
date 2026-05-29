# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `vorch-service` | Module rename (`main` → real path); split into `cmd/vorch-service/` and `cmd/porch/`; ansible packages move to `internal/ansible*`; VM packages move to `internal/vm*`; shared packages move to `internal/{amqp,pub,models,vault}`; log redaction; assignment-tracking SQL if missing. |
| `cloud-manager-api` | New `scripts/porch/{install,start,stop,uninstall}-porch-service.py`. **Conditional**: PATCH `FeatureFlagsController` to expose health-write route if Phase 0003 verifies it missing. |
| `workflow-exec` | This deck (jazzy-herring). |

Repos not listed (`vorch-lib`, `cloud-manager-web`, `cloud-manager-mcp`, etc.) will be on the feature branch but skipped by `hv_ship`.

## Branch
`jazzy-herring`

## Initialize (Task 0000)
Workspace was already advanced to `jazzy-herring` by the planning ship of charmed-panda. Confirm:
```
hv_status  deck: "cloud-manager"
```
If somehow not on `jazzy-herring`:
```
hv_next  deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "porch: carve ansible orchestrator out of vorch-service; retire Python worker"
         title:   "porch: carve ansible orchestrator out of vorch-service; retire Python worker"
```
