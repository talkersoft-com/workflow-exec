# Deck: workflow-relocation

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | Remove `WorkflowFolder` from `DeckFile`, add `Repo` to `WorkflowDef`, update `workflowCmd` and `validate.go` |

Repos not listed will be on the feature branch but skipped by hv_ship.

## Branch
`giddy-minnow`

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
         message: "move workflow repo ownership from deck yaml to workflow definition"
         title:   "workflow: repo link moves from deck to workflow definition"
```
