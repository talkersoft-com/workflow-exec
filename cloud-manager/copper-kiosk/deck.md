# Deck: cloud-manager

## Deck name
`cloud-manager`

## Orchestration id (plan codename)
`copper-kiosk`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-web` | Marketplace storefront, Blueprints builder pages, VM-detail provenance + run-chain panel, api client + Redux slices, nav + feature flag |
| `workflow-plans` | Phase 0005 findings doc (`cloud-manager/copper-kiosk/FINDINGS-vorch-status-surface.md`) |
| `workflow-exec` | This orchestration folder (runtime updates: checkboxes, Retro, Results) |

Repos not listed will be on the feature branch but skipped by hv_ship.
**Must NOT change:** cloud-manager-api, cloud-manager-mcp, vorch-lib, vorch-service, porch.

## Branch
TBD — created by Task 0000 at execution time; record the generated name here.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```
(or `hv_init` if repos are on the default branch)

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat: marketplace UI — storefront, blueprint builder, run-chain visibility"
         title:   "Marketplace UI (copper-kiosk)"
         stage:   "exec"
```
