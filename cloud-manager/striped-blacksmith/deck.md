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

## Branch
`striped-blacksmith`

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
```
Deck was already transitioned to `striped-blacksmith` at approval time — Task 0000 verifies state
only; do NOT run hv_init/hv_next unless hv_status shows the deck is not on `striped-blacksmith`.

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat: marketplace backend — blueprints, marketplace API, run sequencer, MCP tools"
         title:   "Marketplace backend (lucky-engineblock / striped-blacksmith)"
         stage:   "exec"
```
