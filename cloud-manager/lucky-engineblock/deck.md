# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | Blueprint/BlueprintPlaybook/BlueprintEvent entities, VM.BlueprintId, migration, seeds, Blueprint+Marketplace controllers/services/DTOs/mappers, BlueprintRunSequencer, `marketplace` feature flag |
| `cloud-manager-mcp` | New `src/tools/marketplace.ts`, profile registration, dist rebuild |
| `workflow-exec` | This orchestration folder |

Repos not listed will be on the feature branch but skipped by hv_ship.
**Must NOT change:** cloud-manager-web (plan 2), vorch-lib, vorch-service, porch.

## Orchestration id (plan codename)
`lucky-engineblock`

## Branch
TBD — created by Task 0000 at execution time; record the generated name here.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a clean feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat: marketplace backend — blueprints, marketplace API, run sequencer, MCP tools"
         title:   "Marketplace backend (lucky-engineblock)"
         stage:   "exec"
```
